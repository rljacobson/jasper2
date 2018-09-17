---
title: "Defining the Wolfram Language Part 1: Finding Operators"
date: 2018-08-16 15:26
cover: assets/images/posts/TelephoneOperators.jpg
MathJax: true
tags: [Computer science, compilers, Mathematica]
layout: post
current: post
class: post-template
subclass: 'post tag-computer-science'
navigation: True
comments: True
author: robert
---

# Finding All Wolfram Language Operators

In this second article, Part 1 of an *n* part series on *Defining the Wolfram Language*, we start getting our hands dirty hunting down every single operator in Mathematica and each operator's linguistic properties. To my knowledge, nobody outside of Wolfram has created such an exhaustive list before.<!--more--> The operator properties we hope to document are arity, affix, associativity, and precedence. In the next article, "[Generalizing PEMDAS: What is an operator?](generalizing-pemdas-what-is-an-operator)," we will define and discuss the signficance of these linguistic properties of operators for programming languages generally. If these terms are foreign to you, do not worry, you will not need to know what these terms mean for this article, but you can read the next article before reading Part 1 without any confusion.

## Data Sources

It is surprisingly difficult to get accurate information about all Wolfram Language operators implemented in Mathematica. There are a few different sources of information about operators and their properties. I list all publicly known sources (as of August 2018) in the table below, the last two of which appear to be described nowhere else on the public internet. We will explore these sources in more detail in the remainder of this section.

| Source | Description |
|--------|-------------|
| [Official Documentation](https://reference.wolfram.com/language/) | The most common operators only. Usually no precedence or associativity. |
| [`WolframLanguageData[]`](https://reference.wolfram.com/language/ref/WolframLanguageData.html) | Only has info for most common operators. For most operators, gives precedence rank, arity, affix. Less often associativity info is apparent. |
| `Precedence[]` | Undocumented built-in function. When provided the long name of an operator, gives precedence. Only works with some operators and is often incorrect. |
| `UnicodeCharacters.tr` | Gives explicit precedence, affix, and associativity for 338 operators that are a single Unicode character outside of ASCII character set. Note that this system file uses a different precedence numbering scheme than other sources and which applies only to the notebook interface. |
| `` `System`Convert` `` ``MLStringDataDump` `` ``$ExtractedOperators `` | Undocumented list of 272 single character unicode operators all but one of which is in `UnicodeCharacters.tr`. A proper subset of the next source, but I include it for completeness. |
| `` System`Convert` `` `` MLStringDataDump` `` `` $Operators `` | Undocumented list of 340 operators—characters only. Includes most operators, even single and multi-character ASCII operators. Excludes Association brackets, box operators, and obscure bracketing operators. |
| `` Internal`SymbolNameQ[] `` | Undocumented function that apparently identifies a string as a "symbol name." Computing the complement of the set of Unicode characters for which `SymbolNameQ` evaluates to True gives a list of operators, including some single-character box operators found in no other list. |

Of the sources in this table, the only ones that are *documented* are `WolframLanguageData`, `UnicodeCharacters.tr`, and (by definition) the official documentation, and `UnicodeCharacters.tr` is only documented in the sense that [it is mentioned exactly once seemingly in passing in the official documentation](https://reference.wolfram.com/language/Notation/tutorial/ComplexPatternsAndAdvancedFeatures.html) (although see below). It is important to understand that no single source contains all operators, and all but `$ExtractedOperators` and (ironically) the official documentation are necessary to obtain all available information about all operators. 

### The Official Documentation

The first place to look for anyone interested in learning about Mathematica's operators is the operator table given in the [Operator Input Forms](https://wolfr.am/wS1xS8bZ) documentation. This table claims to give operator precedence (in the form of "precedence rank"), associativity, and usage information from which arity and affix are apparent. Unfortunately, you will find that this table is sometimes incorrect, woefully incomplete, and certainly outdated. There are several cases of multiple precedence levels being incorrectly represented in the table as a single precedence level. Associativity information for many listed operators is missing. (`TagSet` is right associative, for example.) Finally, nearly 200 operators are either missing or are listed en mass under labels like, "other ordering operators." 

Other pages of the official documentation, while often very helpful for learning Mathematica, usually do not give information about associativity and precedence. Mathematica's documentation, in my opinion, is generally superlative and sets the standard by which all other documentation should be measured. Even so, it contains a great number of errors and is simply not intended to stand in for the kind of language spec needed by, e.g., compiler tool authors that is motivating our study.

### The Undocumented `Precedence[]` Function

Advanced Mathematica programmers frequently pop up on [mathematica.stackexchange.com](https://mathematica.stackexchange.com/) asking about how to choose a Mathematica operator without a built-in meaning (of which there are many) that has just the right operator precedence they need for their use case. The answer given by the Mathematica gurus is almost invariably to use the undocumented `Precedence[]` function. I am sorry to report that on this point the Mathematica gurus are giving bad advice. Here are a few problems with the `Precedence[]` function:

1. It takes the (symbolic) names of operators, not symbols. For example, `Precedence[Element]` evaluates to `250.`, but `Precedence[∈]` predictably fails with a syntax error. Consequently, you need to know the symbolic name of the operator you are interested in.<sup style="font-weight:bolder" id="a1" name="a1">[1](#opnames)</sup>
2. As a corollary to (1), if an operator *has no name*, `Precedence` is useless. This is not an obscure problem, as there are several important operators with no apparent name, such as the square brackets operator for function application `f[e]`.<sup style="font-weight:bolder" id="a1" name="a1">[1](#opnames)</sup>
3. `Precedence` fails to return a true precedence level for roughly 10% of the operators that do have names. `Precedence` indicates failure by returning `670.`, which is inconvenient since `670.` is a valid precedence level.
4. The same symbolic name can be associated to both the unary and binary operators associated to the function. For example, `Precedence[PlusMinus]` gives the precedence of the unary operator `±x` (`310.`) rather than the precedence of the binary operator `x ± y` (`480.`)
5. `Precedence` combines precedence levels that should be split—but not as much as the [Operator Input Forms](https://wolfr.am/wS1xS8bZ) documentation.
6. `Precedence` is sometimes just plain wrong. For example, contrary to `Precedence`, `Union` has a higher precedence than `Span`.


We have spent a lot of ink explaining why the official documentation, the [Operator Input Forms](https://wolfr.am/wS1xS8bZ) table in particular, and the `Precedence[]` function are not the best sources of information about Mathematica operators in part to correct the record on the subject, as these two sources are what come up the most often in technical discussions, even among experts. But we have made only meager progress toward our goal of accurately documenting every Mathematica operator. Where *should* we—and the gurus on [mathematica.stackexchange.com](https://mathematica.stackexchange.com/)—be looking if not to these sources?

For most purposes, we should look to [`WolframLanguageData[]`](https://reference.wolfram.com/language/ref/WolframLanguageData.html).

### `WolframLanguageData[]`

I have found that the precedence rank returned by [`WolframLanguageData[]`](https://reference.wolfram.com/language/ref/WolframLanguageData.html) is more accurate than the `Precedence` function. As a bonus, `WolframLanguageData` gives you really nice operator synonym, usage, and FullForm data, and programmatic access to all kinds of information related to Mathematica functions and their documentation.

{% highlight wolfram %}
{% raw %}
In[1]:= WolframLanguageData["Power", "PrecedenceRanks"]

Out[1]= {{expr1\_expr2\%expr3, Power[Subscript[expr1,expr2],expr3]}->8,
        {expr1^expr2, Power[expr1,expr2]}->21,
        {expr1^expr2, Power[expr1,expr2]}->21,
        {expr1^expr2, Power[expr1,expr2]}->21,
        {Subsuperscript[expr1, expr2, expr3],
            Power[Subscript[expr1,expr2],expr3]}->21,
        {expr1\^expr2\%expr3, Power[Subscript[expr1,expr3],expr2]}->21,
        {\@expr\%n,Power[expr,Power[n,-1]]}->22}
{% endraw %}
{% endhighlight %}

Another bonus of `WolframLanguageData` is that you do not already have to know the name of every function with an operator. You can ask `WolframLanguageData` to just give you everything in its database, which includes the box sublanguage. The following will generate a list for you. Be warned that it will download data for all 5,000+ functions and ultimately give you a list of 135 (as of August 2018) `{name, rankdata, shortnotation}` triples:

{% highlight wolfram %}
{% raw %}
In[1]:= Select[
            WolframLanguageData[
                WolframLanguageData["Entities"], 
                {"Name", "PrecedenceRanks", "ShortNotations"}
            ], 
            #[[2]] =!= Missing["NotApplicable"] &
        ]
{% endraw %}
{% endhighlight %}

The `"ShortNotations"` data provided by `WolframLanguageData` leaves a lot to be desired, and while arity and affix are usually apparent from the usage data, `WolframLanguageData` usually does not provide enough information to deduce operator associativity. Unfortunately, `WolframLanguageData` will not give you every operator, either. However, this is the *only* source outside of the documentation itself that gives all multicharacter box operators. With some effort, it is possible to use `WolframLanguageData` to programmatically extract even more information about operators from the documentation, but the information earned by this effort is much easier to obtain elsewhere.

Let's dive deeper.

### `UnicodeCharacters.tr`

The Mathematica system files include a file named `UnicodeCharacters.tr` that appears to contain information about how the Mathematica frontend interprets ~1100 Unicode characters. As its name implies, the `UnicodeCharacters.tr` file appears to only include characters that are *single* Unicode characters *outside* of the ASCII character set. It does not include `!` or `/@`, for example. Most people describe the file as "undocumented," but in fact [it is mentioned exactly once in the official documentation](https://reference.wolfram.com/language/Notation/tutorial/ComplexPatternsAndAdvancedFeatures.html) for the Notation Package:

> "The `SyntaxForm` option value can be any operator string valid in the Wolfram Language, that is, any operator contained in the UnicodeCharacters.tr file."

(As an aside, this line in the documentation is incorrect in the sense that `UnicodeCharacters.tr` does not include every operator string valid in the Wolfram Language, ASCII operator strings in particular, while these missing operator strings are still valid values for the `SyntaxForm` option.)

Most of the records in the file are *not* operators, but for the operators that are included, it gives us the most detailed syntactic information of any source. Each record has up to 9 columns (fields) that we might label as follows:

1. Hex Code
2. Long Name
3. Alternative Input Methods
4. Class/Affix
5. Precedence 
6. Associativity
7. Left Spacing
8. Right Spacing
9. Rendered As

This is everything we could hope to know about the linguistics of an operator—the proverbial holy grail of operator syntax. Be aware that it uses a *different* precedence numbering scheme from `Precedence[]`, so be sure not to mix up the two. Note also that the precedence in the `UnicodeCharacters.tr` refers to precedence within the frontend only. (The meaning of this last sentence will become clear in the section, "Programmatically determining properties of operators.")

This code snippet reads the data from `UnicodeCharacters.tr` into the variable `chars`:<sup style="font-weight:bolder" id="a2" name="a2">[2](#mrwizard)</sup>

{% highlight wolfram %}
{% raw %}
In[1]:= chars = ReadList[
            System`Dump`unicodeCharactersTR, 
            Word, WordSeparators -> {"\t\t"},
            RecordLists -> True
        ];
{% endraw %}
{% endhighlight %}

We can filter out non-operators using the Class/Affix column. Let's look at the unique values in that column.

{% highlight wolfram %}
{% raw %}
In[1]:= Union[Select[chars, Length[#] >= 4 &][[All, 4]]]

Out[1]= {"Alias", "Auto", "Close", "CompoundPrefix", "Infix", 
        "InfixOpen", "LargeOp:Prefix", "Letter", "NonBreakingAfterLetter",
        "NonBreakingBeforeLetter", "Open", "Postfix", "Prefix", "WhiteSp"}
{% endraw %}
{% endhighlight %}


These are the ones we want:

{% highlight wolfram %}
{% raw %}
In[1]:= affixes = {"Close", "CompoundPrefix", "Infix", "InfixOpen",
                "LargeOp:Prefix", "Open", "Postfix", "Prefix"};
        ops = Select[chars, Length[#] >= 4 && MemberQ[affixes, #[[4]]]&];
        Length[ops]

Out[1]= 337
{% endraw %}
{% endhighlight %}

The character `〚` is the only one listed with affix `InfixOpen`, while the other matchfix (circumfix) operators are under `Open` or `Close`. Most of them appear to function like parentheses, but their rendering in the frontend is flakey. We also have `LargeOp:Prefix` operators like ∑ (`\[Sum]`), and then ∇ (`\[Del]`), which is the only operator with affix `CompoundPrefix`. 

Of the sources of operators and their precedence, the `UnicodeCharacters.tr` is among the largest, and since it is presumably used by Mathematica internals, you can be sure that it is also the most accurate.

Mysteriously, however, it nonetheless seems to be wrong about precedence in at least one case that I have found so far: `UnicodeCharacters.tr` gives `CircleDot` and `PermutationProduct` the same precedence, but `CircleDot` does have a *higher* precedence than `PermutationProduct` while still being lower than the next highest operator, `SmallCircle`. That is to say, at least in v11.3, `CircleDot`'s real precedence is `606.`, not `605.` as it is listed.

This, my dear readers, is where everyone else who has come this way has stopped looking. Let's go deeper.

### `` System`Convert`MLStringDataDump`$Operators ``

The undocumented<sup style="font-weight:bolder" id="a3" name="a3">[3](#undocumented)</sup> system variable `` System`Convert`MLStringDataDump`$Operators `` is a list of 340 operators without any additional information. The list is only available in the notebook frontend. The undocumented function `` System`Convert`MLStringDataDump`OperatorQ[] `` works by checking membership in this list. 

This is the only list, excluding the documentation and `WolframLanguageData`, that includes all single and multi-character ASCII operators. As with most other sources, no box operators are included. It also excludes Association brackets, obscure bracketing symbols, many LargeOp:Prefix operators, and a handful of other operators found in `UnicodeCharacters.tr`.

We have scoured the documentation, taken a deep dive into Mathematica system files, and explored undocumented features described nowhere else outside of this article save the offices of Wolfram. It may surprise you, then, that there are more operators than we have found so far.

### The Complement of `` Internal`SymbolNameQ `` Characters

Our strategy in this section is to make a list of every Unicode character in the character range used by Mathematica that is *not* a letter, [letter-like form](https://reference.wolfram.com/language/tutorial/LettersAndLetterLikeForms.html), or member of `` System`Convert`MLStringDataDump`$Operators ``, which we asign the shorter alias `ops`. Using the definition of `LetterLikeQ[]` along with the opaque built-in function `` Internal`SymbolNameQ[] ``, we can search the list of Unicode characters to find the complement of the set of characters we can already categorize. Our search criteria is as follows:

{% highlight wolfram %}
{% raw %}
In[1]:= crit[c_] := ! Internal`SymbolNameQ[c] && ! DigitQ[c] &&  ! 
        StringMatchQ[c, WhitespaceCharacter] && ! MemberQ[ops, c];
{% endraw %}
{% endhighlight %}

The following code produces a table of code/character pairs matching the criteria above:

{% highlight wolfram %}
{% raw %}
In[1]:= Select[
            Table[{i, FromCharacterCode[i]}, {i, 65535}], 
            crit[#[[2]]]&
        ]
{% endraw %}
{% endhighlight %}

The result is a bit of a surprise: We get a seemingly random handful of ASCII characters, Association brackets, several of the `LargeOp:Prefix` operators that are missing in the other lists along with a small handful of other Unicode operators also from `UnicodeCharacters.tr`, and, most interestingly, single character versions of multicharacter box operators that are not in any other source. These box operators are:

{% highlight text%}
{% raw %}
0xf7c0: \), 0xf7c1: \!, 0xf7c2: \@, 0xf7c5: \%, 
0xf7c6: \^, 0xf7c7: \&, 0xf7c8: \*, 0xf7c9: \(, 
0xf7ca: \_, 0xf7cb: \+, 0xf7cc: \/, 0xf7cd: \`
{% endraw %}
{% endhighlight %}

The list also includes the nonoperator characters `0xf767: \[ErrorIndicator]`, `0x2043: \[SkeletonIndicator]`, the ASCII delete character `0x007f` (127), the ASCII bell character `0x0007`, and `0xf3ad`, which is an unassigned noncharacter Unicode codepoint "for private use."

Since every other Unicode character is either already identified as an operator or in a set known to exclude operators, any unidentified single character Unicode operators are guaranteed to be in this list. We have found them all. 

### Other "Operators"

There are other *lexemes*—meaningful strings in the language—that could be considered operators but that do not appear outside of the documentation:

* String-related characters like `"` and escape characters `\n`, `\t`, etc.
* The character representation operators: `\[name]`, `\:nn`, and `\.nnnn`.
* The number representation operators: `^^`, `*^`, and ``` `` ```, though the single `` ` `` character does appear in the previous subsection.
* The comment matchfix operator `(*...*)`.

These lexemes are such basic units of the language that they may be processed by Mathematica during or even prior to the lexical analysis phase of the code parsing process similar to how `C/C++` compiler drivers preprocess `#define`, `#include`, and other preprocessor directives before sending the result to the compiler. 

These sorts of language elements are not unique to Wolfram Language, of course, and language tool authors generally make choices about how to handle them based on their syntactic role within the language and what simplifies the software engineering task.

<hr>

<a name="opnames">[1]</a>: <span style="font-size:85%">While this may be a minor complaint, the symbolic names of operators do not always correspond to the function of that name. For example, `Infix` is the symbolic name for the ternary operator `a~f~b` which evaluates as `f[a, b]`, whereas the `Infix[]` function is just a function that effects the display of `f[a, b]`, writing it in terms of the `Infix` ternary operator. Meanwhile, the square brackets operator for function application `f[e]` is apparently nameless despite the existence of the [`Construct[]`](https://reference.wolfram.com/language/ref/Construct.html) function. </span> [↩](#a1)

<a name="mrwizard">[2]</a>: <span style="font-size:85%">
I have borrowed this code with minimal modifications from Mathematica Stack Exchange user [Mr. Wizard](https://mathematica.stackexchange.com/a/153753/27662).
 </span> [↩](#a2)
 
<a name="undocumented">[3]</a>: <span style="font-size:85%">
In fact, the only mention of this variable on the public internet appears to be this article.
 </span> [↩](#a3)