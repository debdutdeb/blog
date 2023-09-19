---
title: "Docker Volumes Permissions Demystified"
date: 2023-09-19T22:51:11+05:30
draft: true
<!-- tags: docker,sysadmin -->
---


Docker and its volumes are one of the most confusing things for most newcomers. New to system administration, new to Linux in general. The most common PITA points that I have seen users experience is "could not create directory: Permission denied" or "could not change ownership: Permission denied". How do we go about investigating and fixing these issues? I'm hoping this article will help on, in most part, investigating the root cause of the issue. Fixing it, will come rather easily afterwards.

> Quick note: this article is valid only for environments with DAC (Discretionary Access Control), MAC (Mandatory Access Control, e.g. SELinux) is out of the scope of this article.

# What do we need before?

Where does any permission denied error start? It's the user. Or rather the user id. 
1. A process tries to modify some file or record of a file on disk, like deleting a file or changing ownership
    <!-- nnop -->
2. The process has an effective user id (try `echo $EUID`) - think of that user trying to perform the action
    <!-- nnop -->
    A process can change it's user in the middle of operation, which changes the value of EUID - the effective user id, but leaves UID intact.
3. The OS checks now the permissions on the file - permission bits  
    Run the following command
    ```sh
    stat -c %A /bin/echo
    ```
    You should see an output such as 
    ```sh
    colima:~$ stat /bin/echo  -c %A
    lrwxrwxrwx
    ```
    Let's break it down a bit - 

    `l` says `/bin/echo` is a link - you can verify that by running `ls -l /bin/echo`
    ```sh
    colima:~$ ls /bin/echo -l
    lrwxrwxrwx    1 root     root            12 Sep 19 16:44 /bin/echo -> /bin/busybox
    ```
    In my case, since I'm using alpine, echo is a symlink to the busybox binary. But in your case it probably won't be the same, especially if, for example if you're using Ubuntu.
    In your case, if it's not a symlink, instead of a `l` it should just be a hyphen like `-`.

    Next, take the whole string and divide into three equal parts. Each part from left to right, defines the permission for the **owning user** of the file, **owning group** of the file, and finally permissions for **any other user/group** on the system.

    | owning user | owning group | other users/groups |
    |----| --- | -- |
    | rwx | rwx | rwx |

    | permission | description |
    | -- | -- |
    | r | read [the file] |
    | w | write [to the file] |
    | x | execute [the file] |

    So, let's write the whole expression in English, `lrwxrwxrwx` - 

    > Check the owner user and group using `stat -c %U-%G /bin/echo`
    > > colima:~$ stat -c %U-%G /bin/echo
    > >
    > > root-root

    "The file is a **symlink**, where user **root** *can read the file, write to the file and execute the file*, *any user in group **root***, *can also read the file, write to the file and execute the file*, *any other user can also read the file, write to the file and execute the file*.


This, above is what DAC or Discretionary Access Control is. Access to files is *at discretion* of the file's owner. 


# Let's talk volumes now

Ok now that we have a good enough understanding of how simple access control works in Linux, next, let's migrate over to the star of the show, docker volumes.

When I speak of volumes, I really speak of both, actual docker volumes, ones you create with `docker volume create` and also bind mounts, you add with `docker container run --volume $PWD/somepath:/somepath` for example. 

## Bind mounts

Bind mounts are rather intuitive. You're *sharing* a directory from host to your container.

Bind mounts are not a docker specific feature. In other words if you're thinking docker watches the folder and copies the files to the container everytime there's a change (you know who you are), you're thinking wrong. That's not how bind mount works.

A bind mount is a Linux feature that you can access directly, without using docker whatsoever.

A bind mount is basically sharing the same location on your disk, from different locations or directories. In other words, it *binds* one directory onto another.

```sh
mkdir /tmp/{dummy,muddy}
sudo mount --bind /tmp/dummy /tmp/muddy
```

The commands above creates two folders `dummy` and `muddy`, then *binds* `dummy` and `muddy` together. 

If you create a file in `dummy`, it will also show up in `muddy`

```bash
colima:~$ sudo mount --bind /tmp/dummy/ /tmp/muddy/
colima:~$ ls /tmp/muddy/
colima:~$ ls /tmp/dummy/
colima:~$ touch /tmp/dummy/file
colima:~$ ls /tmp/dummy/
file
colima:~$ ls /tmp/muddy/
file
```

Docker does the same thing when you use something like `-v $PWD/data:/data`.

## Local volumes

What happens when you run `docker volume create dummy`? It creates a volume named `dummy`. But what is that?

Each volume is a directory. Once the directory is created, it follows the same path as a bind mount would.

The directory is created under volume root, which by default is `/var/lib/docker/volumes`.

So the exact path to the `dummy` volume would be `/var/lib/docker/volumes/dummy/_data`.


## Permissions?

Yes, right. I'm supposed to be focused on permissions. That's what I'm getting at.

The first thing to learn from what we've read here, yet, is volumes are also directories under the hood, so the same access control rules apply for volumes, that apply for bind mounts.

This brings me to talking about process user ids. 

I mentioned previously, each process is run under a user id. The same is true for a docker container. This is either defined in the image's Dockerfile with the `USER` instruction like the following
```dockerfile
USER 1001
```
or, through the command line flag `--user` or the equivalent compose key `user:`. But for this to work, the user must exist in `/etc/passwd` in the container's filesystem, exactly like it should if you were trying to log into a server using ssh. If the user actually didn't exist, server wouldn't allow log in either. This is also pretty easy to check - e.g. `docker run --rm ubuntu cat /etc/passwd`.

---
Alright, so we now know a volume and bind mount, both are directories, therefore a specific set of permissions for different users, and each container's processes, trying to read/write to those mount points, will need to have the right set of permissions applied to those directories for them to work as expected.
---

Let's go over a hypothetical example. Let's take the `ubuntu:latest` image as an example. Run the following in a terminal

```bash
docker run --rm ubuntu id
```

This will show the user id the container will run as. 

```bash
colima:~$ docker run --rm ubuntu id
uid=0(root) gid=0(root) groups=0(root)
```

Since, from the output above, the ubuntu container runs as root, it doesn't matter what permission is set on both a directory that is bound, or a "volume" - it can write to all of them. Which makes this not a good candidate for demonstration.

Ok, now let's try to actually reproduce one of those permission denied errors.

1. Create a volume named test `docker volume create test`
2. Create a directory named test `mkdir /tmp/test`
3. Mount the test volume under `/mnt` and write to some file inside that directory from the container, use the `bitnami/mongodb` image.
    ```sh
    docker volume create test
    docker container run -v test:/mnt bitnami/mongodb touch /mnt/somefile
    ```
    Did it succeed? There you go.
4. Now try the same thing above, but instead use the `/tmp/test` bind mount
    ```sh
    mkdir /tmp/test
    docker container run -v /tmp/test:/mnt bitnami/mongodb touch /mnt/somefile
    ```
    Same `touch: cannot touch '/mnt/somefile': Permission denied` error again.

How do you go about troubleshooting this?

First, we already know, if the user for the container is root, it doesn't matter what the final permission on the directories or files are. So if the write is failing, the container must not be running as root? You can check that rather easily with the command

```sh
docker container run --rm --entrypoint id bitnami/mongodb -u
```

Output
```sh
colima:~$ docker container run --rm --entrypoint id bitnami/mongodb -u
1001
```

Ok that's good to know. This makes sense, as in, if the volume directory does not give user id 1001 somehow, the permission to write to that folder, it would fail like that.

First, what is the permission on those "volumes"? Docker creates the volume directories, so that any user other than root (which applies here), can *only read and execute files* from that folder. 

```sh
colima:~$ sudo stat /var/lib/docker/volumes/test/_data -c %A
drwxr-xr-x # r-x
```

What about bind mounts? 

Well they carry the same permission and owner combination that the source (host directory) has. Which means if the host directory doesn't allow any uid other than itself to write to it, the `container run` would fail.

```sh
colima:~$ stat -c %A /tmp/test/
drwxr-xr-x
```

As you can see, it's the same permission as the volume directory.

Now try to change the behaviour. `chmod o+w /tmp/test` should enable other users to write to this directory (`o[ther]+w[rite]` - "others write")

```sh
colima:~$ chmod o+w /tmp/test/
colima:~$ stat -c %A /tmp/test/
drwxr-xrwx
colima:~$ docker container run -v /tmp/test:/mnt bitnami/mongodb touch /mnt/somefile
colima:~$ ls /tmp/test/
somefile
```

Bingo! `touch somefile` worked !

 



