---
title: "Defining the Wolfram Language Part 2: Operator Properties"
date: 2018-09-04 20:26
cover: assets/images/posts/NuclearOperator.jpg
tags: [Computer science, compilers, Mathematica]
layout: post
current: post
class: post-template
subclass: 'post tag-computer-science'
navigation: True
comments: True
MathJax: True
author: robert
---

In this third installment of our *n* part series, "Defining the Wolfram Language," we begin to study the properties, namely the arity, affix, associativity, and precedence, of the Mathematica operators we found in Part 1. If we ended Part 1 proud of our accomplishment—perhaps even a little smug—then we will get reacquainted with our humility in this article.<!--more--> 

For a refresher on what is meant by the words operator, arity, affix, associativity, and precedence, see [Generalizing PEMDAS: What is an operator?](generalizing-pemdas-what-is-an-operator)

## Programmatically determining properties of operators

Putting these different sources together gives you a database, but there are still some gaps in the dataset. As far as I know, though, these are the only operators anyone who isn't an employee of Wolfram knows about. 

Is there a programmatic way to fill in the gaps in the resulting dataset once you know the operators, that is, what lexical tokens make up the operators? It's easy to programmatically determine affix (prefix, infix, etc.) and arity (unary, binary, etc.) from the usage data that can be obtained from `WolframLanguageData`, and we have everything we need for those operators listed in `UnicodeCharacters.tr`. 

Precedence is a lot harder. I have implemented a strategy that compares two operators by instantiating expressions involving the two operators and inspecting the result. The remainder of this section gives just a few examples to illustrate the fundamental reason why this strategy cannot work.

Two synonyms of the same operator:

{% highlight wolfram %}
{% raw %}
In[1]:= FullForm[Hold[a\[Conditioned]b<->c]]
In[2]:= FullForm[Hold[a\[Conditioned]b\[TwoWayRule]c]]

Out[1]= Hold[Conditioned[a,TwoWayRule[b,c]]]
Out[2]= Hold[TwoWayRule[Conditioned[a,b],c]]
{% endraw %}
{% endhighlight %}

The *only* binary operator that cannot be parsed like this:

{% highlight wolfram %}
{% raw %}
In[1]:= a \[DirectedEdge] b \[UndirectedEdge] c
    
Out[1]= Syntax::tsntxi: "a\[DirectedEdge]b\[UndirectedEdge]c" is incomplete; more input is needed.
{% endraw %}
{% endhighlight %}


This next one is not uncommon:

{% highlight wolfram %}
{% raw %}
In[1]:= FullForm[Hold[a \[LeftTee] b \[UpTee] c]]
In[2]:= FullForm[ToExpression["Hold[a \[LeftTee] b \[UpTee] c]"]]
    
Out[1]= Hold[LeftTee[a,UpTee[b,c]]]
Out[2]= Hold[UpTee[LeftTee[a,b],c]]
{% endraw %}
{% endhighlight %}

There are several differences between the notebook and command line interfaces, and `ToExpression` appears to precisely mirror those differences. Perhaps the command line and `ToExpression` have the same parser. Whatever the case, we cannot assume that a piece of code will execute the same way on the command line as it will through the notebook interface.

Many operators do not have a `FullForm` (unevaluated interpretation) equal to the functions they represent. Consider `Divide`:

{% highlight wolfram %}
{% raw %}
In[1]:= (* Give the Head of the first element within Hold. *)
        Hold[a/b][[1, 0]]
    
Out[1]= Times
{% endraw %}
{% endhighlight %}

Considering this one fact alone, whatever strategy one uses to inspect an instantiated expression to determine precedence will have to account for every special case of how the `FullForm` of each operator might interact with that of another in a nongeneric way. The amount of manual labor one has to do to account for this is $Ω(n^2)$, where $n$ is the number operators.

Then there is the case of `GreaterSlantEqual` and `LessSlantEqual`, both the symbols and their corresponding functions. The short version is, these two symbols are now disassociated from their corresponding functions, which functions now have no operator notation despite the insistance of the documentation. To be clear, I think it's the right move to remap the `*SlantEqual` symbols to `>=` and `<=`, but...  

## Wolfram Language as Language

We have seen that, even in a single version on a single platform, Mathematica itself does not always interpret Wolfram Language consistently in several dimensions:

1. The notebook interface versus `ToExpression` and command line
2. Operator synonyms
3. With respect to the documentation
4. Associated (defining) functions of each operator versus how the input is immediately (re)interpreted; possibly equivalently, an expression's `FullForm` versus the literal interpretation of the expression as an application of its associated (defining) function

