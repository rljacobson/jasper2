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

In this third installment of our *n* part series, "Defining the Wolfram Language," we begin to study the properties, namely the arity, affix, associativity, and precedence, of the Mathematica operators we found in [Part 1](defining-the-wolfram-language-part-1-finding-operators). If we ended Part 1 proud of our accomplishment—perhaps even a little smug—then we will get reacquainted with our humility in this article.<!--more--> 

For a refresher on what is meant by the words operator, arity, affix, associativity, and precedence, see [Generalizing PEMDAS: What is an operator?](generalizing-pemdas-what-is-an-operator)

## Programmatically determining properties of operators

We described several sources of data about Wolfram Language operators and their linguistic properties in [Part 1]([Part 1](defining-the-wolfram-language-part-1-finding-operators)). Putting these different sources together gives you a database, but there are still some gaps in the dataset. As far as I know, though, these are the only operators anyone who isn't an employee of Wolfram knows about. 

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
5. System-internal descriptions of operators like `UnicodeCharacters.tr` and `` System`Convert`MLStringDataDump`$Operators ``.

To be clear, most inconsistencies in interpretation boil down to details of implementation choices that do not affect the output computed by a given expression. Whether `a-b` is parsed as `Subtract[a, b]` or `Plus[a, Times[-1, b]]` generally makes no difference in the final output. Other inconsistencies are a natural consequence of the fact that Mathematica is one of the most complex software systems ever created. The inconsistencies that are genuine bugs are almost all confined to recently introduced operators and outdated documentation. The differences in operator precedence between the notebook interface and `ToExpression` and command line interface are more serious. However, consider:

> For the front end, however, a significant amount of specialized code is needed to support each different type of user interface environment. The front end contains about 700,000 lines of system‐independent C++ source code, of which roughly 200,000 lines are concerned with expression formatting. Then there are between 50,000 and 100,000 lines of specific code customized for each user interface environment.<sup style="font-weight:bolder" id="a1" name="a1">[1](#notes)</sup>

The language evolves just like any natural language evolves. There is no single Wolfram Language just as there is no single English language. We find ourselves with a curious postmodern synthesis of axiomatic mathematical systems and fluid human communication. Our hypothetical quest to lock down the syntax and semantics of Wolfram Language can only ever be Wolfram Language-*like*, not only because of practical challenges of extracting accurate language information from Mathematica, or because a language spec will need to draw an arbitrary line between core language and auxiliary or incidental library content, or even because a bug-for-bug, quirk-for-quirk emulation of Mathematica's behavior is impractical to achieve, but rather because the object of our study, Wolfram Langauge, is itself a thriving living organism changing and adapting to its environment, developing branches on which built-in symbolics like [`Black`](https://reference.wolfram.com/language/ref/Black.html) are eschewed in favor of strings like [`"EntityCount"`](https://reference.wolfram.com/language/ref/EntityValue.html), tendrils where option patterns are replaced with optional arguments or even Free-form Input, appendages where textual expression representations are replaced with opaque graphical object representations like the instantiation of [`LinearLayer[5]`](https://reference.wolfram.com/language/ref/LinearLayer.html#352245294).

Stephen Wolfram has written about how decisions are made for the design of Wolfram Language:

> I’ve worked very hard over the past 30 plus years to maintain the unity and coherence of the Wolfram Language. But every day I’m doing meetings where we decide about new things to be added to the language—and it’s always a big challenge and a big responsibility to maintain the standards we’ve set, and to make sure that the decisions we make today will serve us well in the years to come.

> It could be about our symbolic framework for neural nets. Or about integrating with databases. Or how to represent complex engineering systems. Or new primitives for functional programming. Or new forms of geo visualization. Or quantum computing. Or programmatic interactions with mail servers. Or the symbolic representation of molecules. Or a zillion other topics that the Wolfram Language covers now, or will cover in the future.

> What are the important functions in a particular area? How do they relate to other functions? Do they have the correct names? How can we deal with seemingly incompatible design constraints? Are people going to understand these functions? Oh, and are related graphics or icons as good and clear and elegant as they can be?<sup style="font-weight:bolder" id="a2" name="a2">[2](#design)</sup>

[Stephen livestreams many of his language design meetings](https://www.stephenwolfram.com/livestreams/). It is fascinating to watch the language evolve in realtime and the designers identify and wrestle with often very subtle but important design challenges. What is clear is that Wolfram Language is designed very differently from other programming languages, and I think this difference is precisely because other langauges, generally speaking, are intentionally designed with limitations of expressibility. As a corollary, most successful programming languages change only very slowly or else stand completely still. 

Or, from another point of view, Wolfram Langauge embraces the principle of *orthogonality*, that there should only be a small number of independent fundamental langauge features that are sufficient to express all other aspects of the language, illustrated by the differences between Lisp-like langauges and, say, C++.

>  This is
one great advantage of Lisp-like languages: They have very few ways of forming compound
expressions, and almost no syntactic structure. All of the formal properties can be covered in an hour,
like the rules of chess. After a short time we forget about syntactic details of the language (because
there are none) and get on with the real issues -- figuring out what we want to compute, how we will
decompose problems into manageable parts, and how we will work on the parts. <sup style="font-weight:bolder" id="a2" name="a2">[3](#scip)</sup>

Computer scientist Alan J. Perlis writes in the forward to [[3]](#scip):

> It would be difficult to find two languages that are the communicating coin of two more
different cultures than those gathered around these two languages [Lisp and Pascal]. Pascal is for building pyramids — imposing, breathtaking, static structures built by armies pushing heavy blocks into place. Lisp is for
building organisms — imposing, breathtaking, dynamic structures built by squads fitting fluctuating
myriads of simpler organisms into place. The organizing principles used are the same in both cases,
except for one extraordinarily important difference: The discretionary exportable functionality
entrusted to the individual Lisp programmer is more than an order of magnitude greater than that to be
found within Pascal enterprises. Lisp programs inflate libraries with functions whose utility transcends
the application that produced them. The list, Lisp’s native data structure, is largely responsible for such
growth of utility. The simple structure and natural applicability of lists are reflected in functions that
are amazingly nonidiosyncratic. In Pascal the plethora of declarable data structures induces a
specialization within functions that inhibits and penalizes casual cooperation. It is better to have 100
functions operate on one data structure than to have 10 functions operate on 10 data structures. As a
result the pyramid must stand unchanged for a millennium; the organism must evolve or perish.

For languages designed in the spirit of Lisp, operator syntax is only window dressing. In fact, in many such languages, the syntax and semantics of operators—or even numeric literals!—can even be redefined by the programmer. This is possible to a limited extent in Wolfram Language, but more importantly, all operators in Wolfram Language are merely syntactic sugar for their `FullForm` counterparts.

## Conclusion

Our journey through the landscape of Wolfram Language operator syntax has been a necessary and instructive part of our hypothetical project of rigorously describing the Wolfram Language. The previous section suggests, however, that operator syntax is primarily an ergonomic layer between the human programmer and the underlying language semantics and behavior. The most important language features are those which transcend a particular operator syntax. If all there were in Wolfram Language were `FullForm` expressions, very little of the language would be lost. 

What would remain? We have so far only concerned ourselves with the surface details of Wolfram Language. We will now turn from the question, "What does it look like?," to the question, "How does it work?" Every programming language requires some notion of data, types, operations, and runtime environment, even if one or more of these catagories is combined with another, as when functions can be treated as data, or concealed from the programmer, as in so-called "untyped" languages. Some language characteristics, such as how names are ascribed and resolved, may involve all four of these notions at once. A useful langauge spec will lay out these language subsystems and describe how they interact. 

The boundary between that which is language definition and that which is "implementation defined," that is, a choice left to the language implementor, will continue to be blurry. Ambiguity and undefined behavior is unavoidable despite its offence to our inner mathematician. But it also gives us freedom to make our own choices about what is correct—or even correct the mistakes of the original language designers.

As I consider the rigorous intellectual work required to map out a mature, serious language like Wolfram Language—there are mathematical and engineering mountains to climb—I am reminded again of the words of Alan Perlis who imagined the perpetual compounding complexity of computer systems and the mathematical and software engineering technologies required to tame them:

> Each breakthrough in hardware technology leads to more massive programming enterprises, new organizational principles, and an enrichment of abstract models. Every
reader should ask himself periodically ‘‘Toward what end, toward what end?’’ — but do not ask it too
often lest you pass up the fun of programming for the constipation of bittersweet philosophy.<sup style="font-weight:bolder">[3](#scip)</sup>

I now adapt Perlis' poetic words as we end: *Invent and fit; have fits and reinvent! We toast the Wolfram Language programmer who pens their thoughts within nests of brackets.*

<hr>

<a name="notes">[1]</a>: <span style="font-size:85%"> "[The Software Engineering of the Wolfram System](https://reference.wolfram.com/language/tutorial/TheSoftwareEngineeringOfTheWolframSystem.html)," from the *Wolfram Language & System Documentation Center*. Retrieved September 4, 2018. This article appears to have been written when Mathematica was on Version 6. It is reasonable to assume the code statistics are more dramatic for Mathematica versions ≥ 11.3. </span> [↩](#a1)


<a name="design">[2]</a>: <span style="font-size:85%"> Stephen Wolfram, "[What Do I Do All Day? Livestreamed Technology CEOing](https://www.wired.com/story/what-do-i-do-all-day-livestreamed-technology-ceoing/)," *Wired*, November 11, 2017. Retrieved September 4, 2018. </span> [↩](#a2)

<a name="sicp">[3]</a>: <span style="font-size:85%"> Harold Abelson, Gerald Jay Sussman, and Julie Sussman, *The Structure and Interpretation of Computer Programs*, Second Edition (1996), MIT Press. <br>This classic programming text has introduced generations of programmers and computer scientists to the "different cultures" surrounding Lisp-like languages and Pascal-like languages—and to the computer science common to both. </span> [↩](#a3)
