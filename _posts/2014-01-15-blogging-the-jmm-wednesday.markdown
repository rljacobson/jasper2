---
title: "Blogging The JMM: Wednesday"
date: 2014-01-15 20:53
comments: true
cover: assets/images/posts/PresentationRoom.jpg
MathJax: true
tags: [Mathematics]
layout: post
current: post
class: post-template
subclass: 'post tag-mathematics'
navigation: True
comments: true
author: robert
---

Today I learned about connections between musical rhythm and knot theory, abstract algebra and dance, a Navy ship from the mid 1800s called the U.S.S. Constellation, and some great undergraduate real analysis pedagogy. In today's post, I'll share with you some of the most interesting mathematical ideas I heard today.<!--more-->

Because I will be teaching first semester real analysis this semester, I spent a lot of time in the MAA Session on Topics and Techniques for Teaching Real Analysis. I loved Judit Kardos's talk "Constructing continuous functions" in which she describes how she teaches her analysis students to construct continuous functions with pathological properties. Can you construct a continuous function on $[0,1]$ for which every point in its range has an infinite preimage? Or a monotonic nowhere differentiable continuous function with finite $$\sigma$$-finite length? Kardos's technique is to use the fact that a function on $[0,1]$ is continuous if and only if its graph is compact. She then intersects a sequence of compact sets with fractal-like properties to obtain graphs of continuous functions with bizarre attributes.

My favorite talk from that session is Cesar E. Silva's talk “One Proof Is Not Enough,” in which he reminded us that it can be beneficial for undergraduate real analysis students to see several different proofs of the same result. Silva presented several charming proofs that the real numbers are uncountable ([which he collects in his paper on the arXiv](http://arxiv.org/abs/1209.5119)), some of which are not well known. We all know Cantor’s diagonal argument, but it was not the first proof Cantor published of the uncountability of the reals. The first proof he published goes as follows. Let $$\{w_k\}_{k=1}^\infty$$ be a sequence of real numbers in $[0,1]$. Let $$k_1$$ be the smallest $k$ such that $$w_{k_1}$$ is in $(0,1)$. Now let $$k_2$$ be the smallest $k$ with $$k_2>k_1$$ such that $$w_{k_2}$$ is in $(0,1)$. Set $$a_1 = \min(w_{k_1}, w_{k_2})$$ and $$b_1 = \max(w_{k_1}, w_{k_2})$$. Now choose $$a_2 < b_2$$ to be the next elements in the sequence $$\{w_k\}_{k=1}^\infty$$ that are in $$(a_1, b_1)$$. Iterate this process, generating nested intervals $$(a_1, b_1) \supset (a_2, b_2) \supset (a_3, b_3) \supset \cdots$$. The intersection $$\bigcap_{n=1}^\infty [a_n, b_n]$$ of compact intervals is nonempty, but contains no points in $$\{w_k\}_{k=1}^\infty$$. Thus $$\{w_k\}_{k=1}^\infty$$ cannot contain every point in $[0, 1]$, proving that $[0,1]$ is uncountable.

Cantor's second proof also uses the fact that the intersection of compact sets is nonempty and is just as elementary. Call a set perfect if the set is equal to its set of accumulation points. So in particular, the interval $[0,1]$ is perfect. Now suppose (seeking a contradiction) that $P$ is both a countable set $$P=\{p_1, p_2, \ldots \}$$ and that $P$ is perfect. Let $$B_1=(p_1-1, p_1+1)$$. Since $P$ is perfect, $$B_1$$ contains infinitely many points of $P$. Choose $$B_2$$ to be an open interval about one of the infinitely many points in $$P\cap B_1$$ such that $$\overline{B_2}\subset B_1$$ and $$p_2\not \in B_2$$. Iterate this process, choosing $$\overline{B_n}\subset B_{n-1}$$ such that $$p_n\not\in B_n$$. Now consider the intersection of compact sets $$\bigcap_n (P\cap B_n)$$. This intersection is nonempty, contains accumulation points of $P$, but does not contain any of the $$p_n$$. This is a contradiction. Thus perfect sets cannot be countable. In particular, $[0,1]$ is not countable.

Silva gives more proofs in his arXiv paper, including a game theoretic proof due to Matthew Baker, which are worth reading. 

It's already late, and I haven't worked out my schedule for tomorrow. That's the thing about the Joint Meetings: you never have enough time to do it all!
