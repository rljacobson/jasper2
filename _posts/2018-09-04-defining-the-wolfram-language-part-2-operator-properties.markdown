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
author: robert
---

In this third installment of our *n* part series, "Defining the Wolfram Language," we begin to study the properties, namely the arity, affix, associativity, and precedence, of the Mathematica operators we found in Part 1. If we ended Part 1 proud of our accomplishment—perhaps even a little smug—then we will get reacquainted with our humility in this article.<!--more--> 

For a refresher on what is meant by the words operator, arity, affix, associativity, and precedence, see [Part 2](http://robertjacobson.herokuapp.com/blog/2018/09/03/defining-the-wolfram-language-part-1-terminology/) of this series.

[The information in this series grew out of my work on FoxySheep, an open source parser for the Wolfram Language. Parts 1 and 2 began life as an answer I submitted to [mathematica.stackexchange.com](https://mathematica.stackexchange.com/a/180033/27662). I try to keep the answer up to date.]


## Programmatically determining properties of operators

Putting these different sources together gives you a database, but there are still some gaps in the dataset. As far as I know, though, these are the only operators anyone who isn't an employee of Wolfram knows about. 

Is there a programmatic way to fill in the gaps in the resulting dataset once you know the operators, that is, what lexical tokens make up the operators? It's easy to programmatically determine affix (prefix, infix, etc.) and arity (unary, binary, etc.) from the usage data that can be obtained from `WolframLanguageData`, and we have everything we need for those operators listed in `UnicodeCharacters.tr`. 

Precedence is a lot harder. I have implemented a strategy that compares two operators by instantiating expressions involving the two operators and inspecting the result. The remainder of this section gives just a few examples to illustrate the fundamental reason why this strategy cannot work.

Two synonyms of the same operator:

    FullForm[Hold[a\[Conditioned]b<->c]]
    FullForm[Hold[a\[Conditioned]b\[TwoWayRule]c]]
>     Hold[Conditioned[a,TwoWayRule[b,c]]]
>     Hold[TwoWayRule[Conditioned[a,b],c]]

The *only* binary operator that cannot be parsed like this:

    a \[DirectedEdge] b \[UndirectedEdge] c
    
>     Syntax::tsntxi: "a\[DirectedEdge]b\[UndirectedEdge]c" is incomplete; more input is needed.

This next one is not uncommon:

    FullForm[Hold[a \[LeftTee] b \[UpTee] c]]
    FullForm[ToExpression["Hold[a \[LeftTee] b \[UpTee] c]"]]
    
>     Hold[LeftTee[a,UpTee[b,c]]]
>     Hold[UpTee[LeftTee[a,b],c]]

There are several differences between the notebook and command line interfaces, and `ToExpression` appears to precisely mirror those differences. Perhaps the command line and `ToExpression` have the same parser. Whatever the case, we cannot assume that a 

Many operators do not have a `FullForm` (unevaluated interpretation) equal to the functions they represent. Consider `Divide`:

    (* Give the Head of the first element within Hold. *)
    Hold[a/b][[1, 0]]
    
>     Times

Considering this one fact alone, whatever strategy one uses to inspect an instantiated expression to determine precedence will have to account for every special case of how the `FullForm` of each operator might interact with that of another in a nongeneric way. The amount of manual labor one has to do to account for this is $Ω(n^2)$, where $n$ is the number operators.

Then there is the case of `GreaterSlantEqual` and `LessSlantEqual`, both the symbols and their corresponding functions. The short version is, these two symbols are now disassociated from their corresponding functions, which functions now have no operator notation despite the insistance of the documentation. To be clear, I think it's the right move to remap the `*SlantEqual` symbols to `>=` and `<=`, but...  

## Conclusion

At the end of the day, there just is no public language specification for Mathematica/Wolfram Language. In a sense, the language is *defined to be* whatever Mathematica does. And what Mathematica does on the command line may differ from what it does in the notebook, which may differ from the behavior of ToExpression and friends, all of which may differ from the documentation, which itself may be inconsistent—and all of this in just a single version (v11.3) on a single platform (macos).

So the answer is, no, you cannot determine precedence programmatically, or any other way, for that matter.
