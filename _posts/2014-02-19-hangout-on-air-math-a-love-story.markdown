---
title: "Hangout On Air - Math: A Love Story"
date: 2014-02-19 19:15
cover: assets/images/posts/IdeaNetwork.png
tags: [Mathematics, Science Communication]
layout: post
current: post
class: post-template
subclass: 'post tag-mathematics'
navigation: True
comments: True
author: robert
MathJax: True
---

The [Mathematics Community on Google+][1] had our second ever Hangout On Air last week. I was joined by [Luis Guzman][2], [Jason Davison][3], and [Amy Robinson][4] in a conversation that ranged from fluid dynamics to applying the mathematics of networks to map our ideas to JPEG image compression. <!--more--> (We shared links live to the [Event Page for the Hangout][5] during our conversation.)

<iframe width="560" height="315" src="//www.youtube.com/embed/neyX-3ZIQ6w" frameborder="0" allowfullscreen></iframe>

Here are a few of my favorite topics we touched on during the hangout.

# The mathematics of networks of people and ideas

Amy described her interest in creating networks to discover hidden relationships - networks of people and networks of ideas. She used [Quid][6] and other technologies to create a network from "30 Moleskine notebooks, 3.5G or 756 voice memos, 6,000 Tweets and gigs of autotune on t-payne (don’t judge)." You can read more about Amy's ambitious project [on her blog][7], including a talk she gave for Quantified Self at Stanford.

If you want to do some social network number crunching yourself, Wolfram Alpha can [analyze your Facebook][8] and compute a sophisticated report of your interactions.

# Navier-Stokes equations

Jason showed us a [wild video][9] demonstrating the behavior of a very low Reynolds number fluid. We can describe the behavior of fluids with the famous [Navier-Stokes equations][10] from fluid dynamics. Since fluids are so common (think of all the reasons you'd want to describe air or water), the Navier-Stokes equations are extraordinarily important.

$$\rho \left(\frac{\partial \mathbf{v}}{\partial t} + \mathbf{v} \cdot \nabla \mathbf{v} \right) = -\nabla p + \nabla \cdot\boldsymbol{\mathsf{T}} + \mathbf{f}$$

Despite it's importance, mathematicians still haven't even figured out if solutions exist. In January, Mukhtarbay Otelbaev, Director of the Eurasian Mathematical Institute of the Eurasian National University, [claimed to have a solution][11] to the Navier–Stokes existence and smoothness problem. Mathematicians are still checking his work. There seems to be some [reason to be skeptical][12] that the proof is correct.

# Millennium Prize Problems of the Clay Mathematics Institute

But if Otelbaev really has solved the Navier-Stokes existence and regularity problem, then the Clay Mathematics Institute will award him a $1 million prize for solving one of seven [Millenium Problems][13]. These famous problems are extraordinarily difficult.

# Fourier series and wavelets

We talked about a beautiful circle of ideas related to [Fourier series][14]. Fourier series are a way of decomposing a signal (think the graph of a function) into sines and cosines.

$$f(x) = \frac{a_0}{2} + \sum_{n=1}^\infty \left[a_n\cos\left(nx\right)+b_n\sin\left(nx\right)\right] $$

This way of representing a signal is incredibly useful. If we truncate the infinite sum, we get an approximation to $f(x)$. We can use this to compress our signal. This is how the JPEG digital image format compresses images, for example.

![Fourier Series Animation][15]

More mathematical readers will notice that we are actually talking about a constellation of ideas involving Fourier series, the discrete Fourier transform, and the fast Fourier transform.

Newer digital image compression technologies (such as the JPEG 2000 standard) use wavelets instead of Fourier series. While sines and cosines are periodic functions, wavelets are localized "bumps." Here's an example:

![Meyer Wavelet][16]

We can reproduce signals by adding up shifts and scales of this bump.

If you'd like to learn more about using Fourier series and wavelets to compress digital images, check out this AMS web essay by David Austin titled, "[Image Compression: Seeing What's Not There][17]."

  [1]: https://plus.google.com/communities/100568607954673744130
  [2]: https://plus.google.com/+LuisGuzmanJr/posts
  [3]: https://plus.google.com/+JasonDavison/posts
  [4]: https://plus.google.com/+AmyRobinson/posts
  [5]: https://plus.google.com/events/cofdgqcm9c2gcjr414qj3pn40f8
  [6]: http://quid.com
  [7]: http://amyrobinson.me/2014/02/01/maps-of-ideas/
  [8]: http://www.wolframalpha.com/facebook/
  [9]: https://www.youtube.com/watch?v=UpJ-kGII074
  [10]: http://en.wikipedia.org/wiki/Navier%E2%80%93Stokes_equations
  [11]: http://science.slashdot.org/story/14/01/11/1715227/kazakh-professor-claims-solution-of-another-millennium-prize-problem
  [12]: http://terrytao.wordpress.com/2014/02/04/finite-time-blowup-for-an-averaged-three-dimensional-navier-stokes-equation/
  [13]: http://www.claymath.org/millennium-problems
  [14]: http://en.wikipedia.org/wiki/Fourier_series
  [15]: https://upload.wikimedia.org/wikipedia/commons/5/50/Fourier_transform_time_and_frequency_domains.gif
  [16]: http://upload.wikimedia.org/wikipedia/commons/e/eb/MeyerMathematica.svg
  [17]: http://www.ams.org/samplings/feature-column/fcarc-image-compression