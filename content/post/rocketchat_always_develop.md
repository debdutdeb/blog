---
title: "Perpetual Rocket.Chat Develop Deployment"
date: 2022-06-08T18:49:18+05:30
draft: false
tags: ["docker", "rocket.chat", "deployment"]
categories: ["devops"]
toc: true
---

Since joining Rocket.Chat last year I've been having to deploy it uncountable times, multiple times in a single day even. One of the most critical things, is needing to check community issues on *latest develop* branch (to make sure it's not already fixed) everytime before funneling to product. So it was a lot of re-deploying every few days.

We do have a `develop` tagged docker image, but to *actually* be on the latest develop I'd need to re-pull it every once in a while, and that doesn't really help much. 

I wanted an instance, that is **always** on the `latest` develop branch.

A solution you might think of is a simple shell script with a systemd service. True. But, luckily, I came across [Watchtower](https://github.com/containrrr/watchtower), and that seemed pretty neat.

So now, I'll be documenting how to keep a **perpetual develop running instance of Rocket.Chat** ;)

> It's not just Rocket.Chat, you can use Watchtower with any OCI image, just make sure the tag you're targeting is not specific to patch (like target `:4.2` instead of `:4.2.3` to actually see a difference).

## Install Docker & Docker Compose

```sh
curl -L https://get.docker.com | sh
```

## Deploy Rocket.Chat

Head over to [this pull request](https://github.com/RocketChat/Docker.Official.Image/pull/160) to grab the compose file (only support compose v2 FYI).

Before deploying, add a label to the `rocketchat` service;

```yaml
com.centurylinklabs.watchtower.enable: "true"
```

This will tell Watchtower to keep track of this service. 

## Deploy Watchtower

Copy the following compose file to a diff location (or save with a diff name?).

```yaml
version: '3.0'

services:
    watchtower:
        image: containrrr/watchtower:1.3.0
        volumes: [/run/docker.sock:/var/run/docker.sock:ro]
        command: --label-enable --cleanup
```

It's a pretty simple template. You can change the tag if you wish. **Do not forget the `--cleanup` arg, or else you'll run out of storage pretty soon :p**.

Now just run `compose [-f <filename?>] up -d` for each template.

Your instance will now be **always** on latest `develop`. 

For more information, read [Watchtower's official documentation](https://containrrr.dev/watchtower/) :)