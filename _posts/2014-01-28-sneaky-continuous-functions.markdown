---
title: "Sneaky Continuous Functions"
date: 2014-01-28 21:41
cover: assets/images/posts/DeceptiveBends.jpg
tags: [Mathematics, Education, Calculus]
layout: post
current: post
class: post-template
subclass: 'post tag-mathematics'
navigation: True
comments: True
author: robert
---
<script src="https://sagecell.sagemath.org/static/jquery.min.js"></script>
<script src="https://sagecell.sagemath.org/static/embedded_sagecell.js"></script>
<script>
// Make any div with class 'sage' a "minimal" Sage cell with "Plot it!" button.
sagecell.makeSagecell({inputLocation:  'div.sage',
                       evalButtonText: 'Plot it!'});
// Make *any* div with class 'compute' a Sage cell.
sagecell.makeSagecell({inputLocation: 'div.compute',
                       evalButtonText: 'Evaluate'});
</script>

While the target audience of this article is my fantastic calculus students, other math teachers might enjoy it as well.

## Sneaky Continuous Functions

When students in first semester calculus first start learning about limits, they are often asked to determine limits using the graph of a function, which we will call the graphical method, and also by constructing a table of values of the function, which we will call the numerical method. Students should be warned that these methods, while perfectly legitimate and often quite useful, are really just fancy ways of *guessing* the value of the limit, that is, the graphical and numerical methods do not supply us with mathematical certainty regarding the value of the limit. After all, what if your function is very sneaky and merely *looks like* it's approaching a value $L$ as $x$ approaches $c$ when in fact it ultimately approaches a different value $K$?<!--more-->

Students quickly learn that a function $f(x)$ is continuous at a point $x=c$ if $$\lim_{x\to c} f(x) = f(c)$$. So limits of continuous functions are very easy to compute: just plug $c$ into the function.

Here we give a very simple construction for *sneaky* continuous functions, that is, continuous functions that look like they approach some value $L$ as $x$ approaches some $c$ but that really approach a different value $K$.

### A Sneaky "Zero" Function

We start by constructing a continuous function that looks very much like the constant zero function but that actually has a "spike" near $x=0$. First, consider this "tent" function $t(x)$ which we plot below:

$$
t(x) = \left\{
  \begin{array}{l l}
    0 & \quad \text{if $x < -1$ or $x>1$}\\
    1 + x & \quad \text{if $-1 \leq x < 0$}\\
    1 - x & \quad \text{if $0 \leq x \leq 1$}
  \end{array} \right.
$$

<div class="sage">
<script type="text/x-sage">
def t(x):
    if x < -1 or x > 1:
        return 0
    elif x >= -1 and x < 0:
        return 1+x
    #else
    return 1 - x

plot(t, -4, 4, thickness=3)
</script>
</div>

This function isn't exactly sneaky. We could easily make a table of $x$ values approaching $x=0$ and see that this function clearly approaches $1$. But how about the function $s(x)=t(10^9x)$? The function $s(x)$ is certainly continuous, and even for values of $x$ very close to $x=0$, the value of $s(x)$ is still zero. In fact, we would have to choose $$\mid x \mid \, < \frac{1}{1,000,000,000}$$ before we'd suspect that $s(x)$ is not the constant function! Our function $s(x)$ is very sneaky indeed. What happens if we plot $s(x)$? Can the computer even tell the difference between $s(x)$ and the constant zero function?

<div class="sage">
<script type="text/x-sage">
def t(x):
    if x < -1 or x > 1:
        return 0
    elif x >= -1 and x < 0:
        return 1+x
    #else
    return 1 - x

s = lambda x: t(10^9*x)
plot(s, -4, 4, thickness=4)
</script>
</div>

Nope! This is the number one lesson of doing math with computers: **COMPUTERS WILL LIE TO YOU.**

### Constructing Other Sneaky Functions

We can use $s(x)$ to construct other sneaky functions. Here's an example of a function that looks like a nice parabola passing through the point $(1, 2)$:

$$
f(x) = x^2 + 1  -5\cdot s(x).
$$

<div class="sage">
<script type="text/x-sage">
def t(x):
    if x < -1 or x > 1:
        return 0
    elif x >= -1 and x < 0:
        return 1+x
    #else
    return 1 - x

s = lambda x: t(10^9*x)
plot(lambda x: x^2 + 1 - 5*s(x), -2, 2, thickness=3)
</script>
</div>

Since $x^2+1$ and $$-5\cdot s(x)$$ are both continuous, their sum is continuous also, so $f(x)$ is continuous. Therefore, $$\lim_{x\to 1} f(x) = f(1) = 2-5 = -3$$. From the graph, it *looks like* $f(x)$ approaches $2$, which definitely is not equal to $-3$. Sneaky!

You can see that we can use this same method to make all kinds of sneaky functions by adding a multiple of $s(x)$ to the function of our choice. Try it yourself in the Sage cell below.

<div class="compute">
<script type="text/x-sage">
def t(x):
    if x < -1 or x > 1:
        return 0
    elif x >= -1 and x < 0:
        return 1+x
    #else
    return 1 - x

s = lambda x: t(10^9*x)
f = lambda x: x^2 + 1 - 5*s(x)
plot(f, -2, 2, thickness=3)
</script>
</div>

