---
title: "Defining the Wolfram Language Part 0: The Challenge"
date: 2018-07-02 20:14
cover: assets/images/posts/ArrivalLogogram.jpg
MathJax: false
tags: [Computer science, compilers, Mathematica]
layout: post
current: post
class: post-template
subclass: 'post tag-computer-science'
navigation: True
comments: True
author: robert
---

*What is the definition of the Wolfram Language?* This is the first in a series of articles attempting to answer this question.<!--more--> 

What *is* a programming language, really? Most programming languages in production use, like C++ or Javascript, have some kind of formal specification that defines the language that each implementation (e.g. each compiler or web browser) must adhere to. Many languages also have a "reference implementation" of the language whose source code could be studied, at least in principle, to determine precisely how the language works. [Wolfram Language](https://www.wolfram.com/language/), the programming language of [Mathematica](http://www.wolfram.com/mathematica/), has neither of these things. 

But what if it did? What would a language specification look like for Wolfram Language? If you were tasked with writing the spec, how would you do it, and what choices would you make? This is how we shall imagine ourselves throughout this series of articles: as language documentarians reconstructing the syntactic and semantic structure of Wolfram Language from its usage.

Some of the information in this series grew out of my work on [FoxySheep](https://github.com/rljacobson/FoxySheep), an open source parser for the Wolfram Language. Parts [1](defining-the-wolfram-language-part-1-finding-operators) and [2](defining-the-wolfram-language-part-2-operator-properties) began life as an answer I submitted to [mathematica.stackexchange.com](https://mathematica.stackexchange.com/a/180033/27662). I try to keep the answer up to date.


## The Impossibility Problem

At the end of the day, there just is no public language specification for Wolfram Language. We might say that the language is *defined to be* whatever Mathematica does. But this "definition" leaves a lot to be desired. If we want to produce a formal language spec from its implementation in Mathematica alone, we are faced with some significant challenges:

* It requires access to a proprietary, expensive piece of software.
* The language evolves from one version of Mathematica to the next.
* It is not clear how to determine—or even discover—some language features using only the language itself. 
* It is labor-intensive. We shall see in Part 1 that Wolfram Language has about 400 different operators, all of which need to be rigorously described.

Most importantly, it is not clear that this "definition" is even self-consistent. (*Spoiler alert: It isn't.*) In other words, what Mathematica does on the command line may differ from what it does in the notebook, which may differ from the behavior of `ToExpression` and related functions, all of which may differ from the official documentation, which itself may be inconsistent—and all of this in just a single version of Mathematica running on a single platform. 

We are also faced with a demarcation problem: Where does the *language* of Wolfram Language end and the various non-essential libraries and APIs that ship with Mathematica begin? Surely, control structures like [`If`](https://reference.wolfram.com/language/ref/If.html) and [`While`](https://reference.wolfram.com/language/ref/While.html) belong to the language, while functions like [`WordCloud`](https://reference.wolfram.com/language/ref/WordCloud.html) and [`WikipediaSearch`](https://reference.wolfram.com/language/ref/WikipediaSearch.html) are clearly not essential language features. But what about [`Plot`](https://reference.wolfram.com/language/ref/Plot.html) or [`HistogramTransform`](https://reference.wolfram.com/language/ref/HistogramTransform.html) or [`TimeValue`](https://reference.wolfram.com/language/ref/TimeValue.html)? Are the mathematical functions like `Sin` and `Integrate` essential features of the language? Some languages include a standard library of functions that is considered part of the language standard. Should there be a standard library for Wolfram Language, and if so, which functions should belong to it?

The demarcation problem extends to what might be considered implementation details. For example, [Mathematica renames "local" variables](https://reference.wolfram.com/language/tutorial/VariablesInPureFunctionsAndRules.html) within an inner scope to unique names to avoid potential name collisions. There presumably needs to be some kind of mechanism to maintain λ calculus semantics for pure functions, but maybe that is an implementation detail rather than part of the hypothetical language spec.

We will have more to say about the technological and philosophical implications of these questions later in this series. 

## Why do we care?

Different from a user manual intended to teach someone how to use the language, a formal specification is a complete, precise technical definition of what makes up a valid program in the language, and, perhaps counterintuitively, is usually not very helpful in learning how to program in the language. The quality of Mathematica's user manual is generally considered among the highest in the industry. What, then, is the use of a formal specification for Wolfram Language? 

First, a formal specification is essential for someone wishing to *implement* the language. This is relevant to only a handful of people in the world who write compiler tools, but the value of the work done by these few people is trememdous, as their tools may be used by many thousands of people.<sup name="a1" id="a1">[1](#intellij)</sup>

Second, it is common for advanced users of Wolfram Language (Mathematica) to want to employ "custom" operators for their own purposes. The language has many operators for a variety of uses, including operators that intentionally have no built-in meaning so that users can use them for their own purposes. New Mathematica users are often puzzled why their code behaves differently than they expect due to incorrect assumptions about the precedence of operators, that is, the order in which the operations are performed. More advanced users often wish to know the precedence and other properties of operators so they can select an operator with no built-in meaning that behaves precisely according to whatever purpose is desired.

Finally, the study of programming languages in general is a rewarding and rich experience that takes one on a veritable tour of nearly every corner of computer science and software engineering. Students of programming languages have much to learn from comparing and contrasting different languages and implementations. Wolfram Language is an interesting language for a variety of reasons: it shares features of LISP and ML yet supports imperative programming; it supports both dynamic and lexical scoping, namespaces (contexts), and modules; it is untyped/weakly typed; and so on.

## Conclusion

The creator of Mathematica Stephen Wolfram and his son Conrad Wolfram were consultants for the film Arrival.<sup name="a2" id="a2">[2](#arrival)</sup> In the film, linguists are faced with the challenge of deciphering an alien language completely unlike any other known language in order to communicate with alien visitors. Stephen and Conrad were asked to imagine what it would look like to try to accomplish this task. For the characters in the film, the task seems at first impossible, but the benefit was potentially tremendous: the sharing of knowledge. 

Rigorously describing a programming language with only a single proprietary implementation is not quite as dramatic, and Wolfram Language is hardly an alien language. Despite the heavy-handedness of the analogy, the benefits of a mutually comprehensible language is the same nonetheless: information can be exchanged. A technical description of Wolfram Language would enable the sharing of knowledge among anyone or anything that can speak Wolfram Language: human beings, compiler tools, third party software, and, importantly, Mathematica itself. 

<hr>

<a name="intellij">[1]</a>:
A great example of this is the [Mathematica plugin for the IntelliJ suite of IDEs from JetBrains](http://wlplugin.halirutan.de/). The Mathematica plugin is essentially the work of one person, Patrick Scheibe, but is enjoyed by tens of thousands of users. [↩](#a1)

<a name="arrival">[2]</a>: Stephan Wolfram, "[Quick, How Might the Alien Spacecraft Work?](http://blog.stephenwolfram.com/2016/11/quick-how-might-the-alien-spacecraft-work/)," blog post, November 10, 2016. [↩](#a2)
