---
title: "Fundamental Theory of Arithmetic"
date: 2024-03-10T22:51:56+05:30
draft: false
math: true
tags: ["mathematics", "number-theory"]
---

> There is one and only one unique prime factorization of a number.

If you have a number, like \\(34\\), factoring it we get -

$$
\begin{align}
fact(34) = 2 * 17
\end{align}
$$

Question is, can you have a different set that is other than \{ \\(2\\) \} It may seem evident from the example above but let’s start with an informal proof.

Firstly, we'll use a different definition of a set, where elements **can show up multiple times** - this is critical in making the proof simpler.

Suppose the set representing all the primes of a number is \\(S =\\) \{ \\(a_1, a_2, a_3, \dots, a_i\\) \} for \\(i \in \mathbb{N}\\) and \\(S \subset \mathbb{P}\\). Let’s also change the number to \\(988\\), something you can’t factor in your head (at least I hope not).

Recalling a prime, *is a number that can not be factored any further*, or better yet *a number that is divisible by 1 and itself*.

This establishes a uniqueness to a prime number, that **it is it**. You can’t replace one prime for another, because they represent a specific count, that can’t even be further broken down in terms of factors.

A number can be represented as a multiple of many primes. Let’s say \\(988\\) is a multiple of \\((a_1, a_2, a_3, \dots, a_i)\\) (same as set \\(S\\)), ordered in ascending. Therefore, if \\(\vert S \vert = i\\) (number of elements in a set, recall, in our definition of a set, we can have duplicates), \\(i\\) is a constant. If there exists another set \\(T\\) such that the number got from multiplying all its elements is \\(988\\), \\(\vert T \vert\\) must also be \\(i\\). Since we can’t break down a prime any further, \\(\vert T \vert\\) can not change, if it changes, we are effectively introducing another prime, and we're no longer talking about \\(988\\) anymore.

Now assume there is a set \\(K\\) that similarly represents the number \\(988\\) in its ordered factors form, but \\(K \ne S\\).

\\(K \ne S\\) indicates there exists at least one element at \\(i\\) such that \\(k_i \ne s_i\\) for \\(k_i \in K\\) and \\(s_i \in S\\).

$$
\begin{align}
\exists k_i \in K \land \exists s_i \in S \vert k_i \ne s_i
\end{align}
$$

Now let's assume further, and reduce the cases to just one. In other words instead of saying "there exists at least one", thus saying "there could be 5 or 10 or 100", we simply say "there is one such an element". And the index of that element is \\(c\\).

Therefore \\(k_c \ne s_c\\) for \\(k_c \in K \land s_c \in S\\).

From our initial assumption of \\(S\\) and \\(K\\) representing the same number, if we take our a number from both of these ordered sets, from the same index, the resulting sets should also represent *a* same number (not \\(988\\) but another number, both of which are the same).

So, effectively we would be saying 

$$
\begin{align}
\frac{988}k_c & = \frac{988}s_c \\\
\frac{1}k_c  & = \frac{1}s_c \\\
k_c & = s_c
\end{align}
$$

This contradicts our initial assesment of \\(S \ne K\\) and establishes that \\(S = K\\). Thus proving the prime factorization of a number is indeed unique.

If you are thinking about what if more than one elements were inequal? The answer is it wouldn't matter. The sets are ordered, two numbers at different indexes can not complement one another, unless they are the same, in which case they are the same as one number differring.

---

Again, this can be proved in our head through intuition. But a partially formal proof sometimes puts our mind at ease.

This theory has a lot of applications. Which I won’t be getting into today. Goal was to prove it “partially formally” - a mix of formal notations and logic, with a hint and approach of informality.


