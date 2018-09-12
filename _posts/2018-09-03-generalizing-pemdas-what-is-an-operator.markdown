---
layout: post
title: "Generalizing PEMDAS: What is an operator?"
date: 2018-09-03 11:30
cover: assets/images/posts/NuclearOperators2.jpg
MathJax: true
tags: [Computer science, compilers]
layout: post
current: post
class: post-template
subclass: 'post tag-computer-science'
navigation: True
comments: True
author: robert
---

In programming languages, an *operator* is a symbol used to represent a specific operation such as subtraction of integers or dereferencing a pointer. The symbols `+`, `*`, and `!` are commonly used as operators in many programming languages, for example. In this article we define some basic programming language terminology that allows us to categorize operators according to their different properties.

## Computation with Expressions

Virtually all programming languages have *expressions*, like `2 + 4`, which consist of *operators* (the `+` operator in `2 + 4`) and their *operands* (or *arguments*; the `2` and the `4`), that is, the things operators operate on.  This language is barrowed from mathematics where, for example, $17/2 + \sin(\pi/2)$ is an expression that *evaluates* to the *value* $19/2$. Most programming lanugages also have other language constructs like statements and declarations that are not expressions. In some languages, like Wolfram Language, [*everything is an expression*](https://reference.wolfram.com/language/tutorial/EverythingIsAnExpression.html). In these languages, an entire program itself is an expression, albeit possibly a very complex one, and executing a program is called *evaluation*.

## Properties of Operators

### Arity

The [arity](https://en.wikipedia.org/wiki/Arity) of an operator is just a fancy word for how many operands (arguments) the operator takes. Here are some common examples:

| Arity | Operator | Example Expression |
|-------|----------|--------------------|
| "Nullary"<br> (takes no arguments)| Constants | 7  |
| Unary  | Factorial: `!` | $3! = 3 \cdot 2 \cdot 1 = 6$ |
| Binary | Addition: `+` | $2 + 4 = 6$ |
| Ternary | The conditional operator <br>found in many languages: <br>`a ? b : c` | `// if a>b then x else y` <br>`result = a > b ? x : y; `|
| Variadic <br>(takes a variable number of arguments) | Lists: `{x, y, ...}` | `{ }` (empty list); `{2, 3, 5, 7, 11}` |
| $n$-ary <br>(takes $n$ arguments) | A function of $n$ arguments: $f(a_1, a_2, \ldots, a_n)$ | `DeleteCases[expr, pattern,`<br>` levelspec, n]` |

We will occassionally abuse the language above by using *variadic* when we mean *n-ary* and vice versa. The Wikipedia article on [arity](https://en.wikipedia.org/wiki/Arity) has a long list of names for different arities.

### Affix

The *affix* of an operator refers to the operator's position within an expression with respect to its operands. Again, some common examples:

| Affix | Operator | Example Expression |
|-------|----------|--------------------|
| Prefix: precedes its operand(s) | Unary minus: `-` | $-7$ |
| Postfix: follows its operand(s) | Factorial: `!` | $3!$ |
| Infix: between its operands | Addition: `+` | $2 + 4$ |
| Matchfix (Circumfix): surrounds its operand(s) | Floor function: $\lfloor{\;}\rfloor$ |  $\lfloor{2.78}\rfloor = 2$ |

The affix of operators with more than two operands defy easy categorization. To illustrate, suppose we wish to invent our own ternary operator, and we have the symbols `@`, `#`, `$`, and `&` available to use to represent our operator if we need them. If `a`, `b`, and `c` are operands, then we could define the operator as `@ a # b $ c &` or `a # b $ c &` or `@ a # b $ c` or `a # b $ c`. Or instead of using different symbols to separate the operands, we could use just one symbol, as Mathematica does with the `Span` operator: `i ;; j ;; k`. 

### Associativity

The *associativity* of an operator determines how an operator *groups* with expressions around it. Consider the exponentiation (power) operator `^` in the expression `2^3^2`$=2^{3^2}$, which is a perfectly valid expression in many programming languages and in mathematics. Setting aside mathematical tradition, this expression *could* mean either `(2^3)^2` $= (2^3)^2 = $ `8^2 = 64`, but it could also mean `2^(3^2)` $= 2^{(3^2)} = $ `2^9 = 512`. These are clearly very different meanings! In the first interpretation, we say the `^` operator *associates to the left*, because the left-most operation is done first. In the second interpretation, we say the `^` operator *associates to the right*. Mathematical tradition and most programming languages define `^` as right associative.

### Precedence

You may recall from elementary school that some mathematical operators must be done before others. In the expression  $2+3\times 5$, for example, we need to multiply the 3 by 5 before we do the addition: $2+3\times 5 = 2+(3\times 5) = 2+15 = 17$. If we fail to follow the correct order of operations, we are likely to get the wrong answer: $2+3\times 5 \neq (2 + 3) \times 5 = 5 \times 5 = 25 \neq 17$. School children learn the correct order of operations by remembering the initialism "PEMDAS" or, "_P_lease _E_xcuse _M_y _D_ear _A_unt _S_ally," which assigns the arithmetic operators the precedence:

1. Parentheses
2. Exponents
3. Multiplication
4. Division
5. Addition
6. Subtraction

(In fact, we usually give multiplication the same precedence as division, and addition the same precedence as subtraction.)  The numbers in the list above are called precedence levels and tell us the order each operation is performed when an expression is evaluated. If we expand our list of operators, we must decide which operations should precede which other operations during evaluation of the expression, that is, what precedence each operator should have.

An equivalent way of thinking of the precedence of an operator is as the "binding power" of the operator. In the expression $2+3\times 5$, we say that the $\times$ operator binds more tightly to nearby expressions (numbers, in this case) than does the $+$ operator, and consequently $2+3\times 5 = 2+(3\times 5)$

## Conclusion

The arity, affix, associativity, and precedence of an operator are essential to know in order to determine which parts of an expression a given operator is acting on and therefore how the expression should be evaluated. Our goal in the next article will be to discover all of the operators that exist in the Wolfram Language and determine the arity, affix, associativity, and precedence of each.