# Another Look at Lexical Analysis

I will describe a few of the latest technologies for lexical analysis. If you are like most programmers, even compiler engineers, you probably have not heard of any *new* algorithms for lexical analysis. The most sophisticated algorithms in common use in compilers today date from the 70s and 80s, over 30 years ago. Let’s first talk about why this is.

## Why lexical analysis is no longer a concern

### The Ivory Tower Versus What Works

The typical undergraduate compiler textbook describes the first steps in constructing a compiler as follows:

1. Implement the McNaughton-Yamada-Thompson algorithm and use it to construct a nondeterministic finite automata (NFA) for the lexical dictionary of the target language.
2. Implement the Rabin–Scott powerset construction algorithm to convert the NFA to a deterministic finite automata (DFA).
3. Implement Hopcroft's algorithm to minimize the DFA.
4. Write an interpreter for the DFA to process the input character stream. 

No compiler engineer does this. Despite its disproportionate treatment in compiler textbooks, lexical analysis has not been a significant component of the design effort or performance characteristics[^lexperf] of modern compilers for at least a couple of decades. Compilers that use finite automata always employ a scanner generator like lex or flex, which does the heavy lifting of Steps 1-4 above. Indeed, the first three steps are entirely generic. Step 4 is dependent on the implementation language, but there are few design decisions to make when authoring a DFA interpreter, so Step 4 is always incorporated into scanner generator tools.

Few mainstream compilers use a finite automata lexing strategy at all. The table below includes the most popular programming languages as of 2020. 

| Compiler                                                     | Scanner Implementation                    | State Machine  | Rationale                                                    |
| ------------------------------------------------------------ | ----------------------------------------- | -------------- | ------------------------------------------------------------ |
| [Clang](http://clang.llvm.org/docs/InternalsManual.html#the-lexer-and-preprocessor-library): C, C++, Objective-C | Hand-coded                                | No             | Performance and complexity; <br />"lexing and preprocessing C source code” is a “nasty process.” |
| [GCC](https://gcc.gnu.org/onlinedocs/cppinternals/Lexer.html) (>=v3.0): C, C++, Objective-C | Hand-coded                                | No             | Performance and complexity                                   |
| [Rust](https://github.com/rust-lang/rust/blob/2917d993023dec5111147a1552ec78b206a5a37e/src/librustc_parse/lexer/mod.rs) | Hand-coded                                | No             |                                                              |
| [Python](https://github.com/python/cpython/blob/master/Parser/tokenizer.c) (CPython) | Hand-coded                                | No             |                                                              |
| [Hotspot](https://github.com/AdoptOpenJDK/openjdk-jdk8u/blob/aa318070b27849f1fe00d14684b2a40f7b29bf79/langtools/src/share/classes/com/sun/tools/javac/parser/JavaTokenizer.java) (Java) | Hand-coded                                | No             |                                                              |
| [V8](https://github.com/v8/v8/blob/master/src/parsing/scanner.cc) (JavaScript) | Hybrid Hand-coded<br>and Custom Generator | No             | Performance                                                  |
| [Spidermonkey](https://hg.mozilla.org/mozilla-central/file/tip/js/src/frontend/TokenStream.cpp) (JavaScript) | Hand-coded                                | No             | Multiple character encodings, performance, complexity        |
| [C#](https://github.com/dotnet/roslyn/blob/master/src/Compilers/CSharp/Portable/Parser/Lexer.cs) (Roslyn) | Hand-coded                                | No             | Performance and complexity                                   |
| [PHP](https://github.com/php/php-src/blob/5d6e923d46a89fe9cd8fb6c3a6da675aa67197b4/Zend/zend_language_scanner.l) | Generated (RE2C)                          | Yes (implicit) | Uses bison                                                   |
| [Go](https://go.googlesource.com/go/+/refs/heads/master/src/go/scanner/scanner.go) | Hand-coded                                | No             | Uses bison                                                   |

Most of the above have internal documentation that discusses the parser but does not even mention the scanner, underscoring the lack of development focus on lexical analysis. 

### Why Handwritten Scanners

The primary reason for writing a scanner by hand is that with a hand-coded scanner the compiler engineer has complete control over lexical analysis. In particular, it is often convenient to blur the boundaries between scanning[^lexterms] and parsing. The C language family is on the outermost extremes of lexical complexity: lexical and semantic analysis are deeply entangled. In fact, the C++ standard describes [six phases](https://en.cppreference.com/w/cpp/language/translation_phases) *before* tokenization! 

Generally, though, scanning accounts for only a small portion of compilation time. Consider Rust compiler developer [Georg Brandl's unscientific benchmark](https://users.rust-lang.org/t/single-pass-languages-and-build-speed/8199/2) for rustc: 

> when compiling regex, a middle-sized crate, in debug mode (with -Z  time-passes), parsing up to building an AST takes 0.098 seconds, while  the LLVM backend alone spends 4.471 seconds.

Increasing the speed of lexical analysis by a factor of 10 would be imperceptible![^amdahl]  Also, a language may have special requirements that are simply not available in existing scanner generator tools. Languages that grow over time often outgrow their original generated scanner specification, making maintaining the generated scanner harder than maintaining a hand-coded scanner.

Scanners generated by a scanner generator tool are much more common with smaller languages and DSL’s where the time and effort required to write a scanner by hand cannot be justified. But doesn’t someone have to write the scanner generator? The scanner generators first written in the 70’s by the researchers who invented the regular expression algorithms are still in wide use today.

## The problem with compiler textbooks

Not only are compiler textbooks out of touch with how contemporary compilers are designed, they are also out of touch with contemporary research in the recognition of regular languages, the theory underlying lexical analysis. This curious phenomenon is almost certainly the fault of the tremendous success of The Dragon Book, officially titled *Compiler: Principles, Techniques, and Tools*. The book began life as *Principles of Compiler Design* by Alfred V. Aho and Jeffrey D. Ullman, sometimes called the Green Dragon Book. The authors reworked the book with the help of Ravi Sethi, changed the dragon to red, and published it under its current title in 1986. After the Red Dragon Book came the Purple Dragon Book, with Monica S. Lam joining the other three authors, in 2006. The book is easily the most cited and used compiler textbook in the world. 

Aho and Ullman invented much of the theory of regular expression algorithms. Aho is the “A” in the Unix utility AWK. Ullman, with Steve Johnson, created the parser generator yacc. A young intern at Bell Labs named Eric Schmidt, with Mike Lesk, implemented Aho’s algorithms to create the scanner generator lex. All of this is to say, there is a reason why the Dragon Books are so important historically. As a technical reference, the latest Dragon Book is very well written and continues to be an excellent text that deserves a place on every compiler enthusiast’s bookshelf.

The Dragon Book set the standard: Finite automata are featured heavily in the Dragon Book, which meant—and continues to mean—that every compiler textbook needs a generous treatment of finite automata, in contradiction to contemporary practice and common sense.

## The Value of Lexical Analysis Today

The theory of regular languages and finite automata *is* important, and I believe its place within the undergraduate computer science curriculum is as justified today as it has ever been. Regular expressions and state machines have a secure place in a course on the theory of computation, and regular languages are certainly important for any course in formal languages. From an applied perspective, state machines, and thus finite automata, continue to be important in a variety of applied contexts.

Today realtime packet inspection drives the research effort in fast finite state machine and DFA algorithms. Intrusion detection/prevention systems (IDS/IPS) require low latency, high throughput analysis of packets traveling through a network. Packet header and payload data are matched against regular expressions to detect of predetermined patterns known to be associated with network attacks. The latency and throughput requirements of IDS’s are so great that the matching algorithms are often implemented in FPGA hardware inside network devices. More efficient algorithms mean cheaper hardware, reduced power consumption, and the ability to match against a larger number of more complicated patterns. 

A number of new techniques have been developed to reduce the memory requirement and increase the runtime of matching input against regular expressions. Despite these developments, compilers and compiler textbooks continue to use algorithms from before I was born.







```java
deprecatedPrefix = false;
// At beginning of line in the JavaDoc sense.
if (bp < buflen && ch == '@' && !deprecatedFlag) {
  scanCommentChar();
  if (bp < buflen && ch == 'd') {
    scanCommentChar();
    if (bp < buflen && ch == 'e') {
      scanCommentChar();
      if (bp < buflen && ch == 'p') {
        scanCommentChar();
        if (bp < buflen && ch == 'r') {
          scanCommentChar();
          if (bp < buflen && ch == 'e') {
            scanCommentChar();
            if (bp < buflen && ch == 'c') {
              scanCommentChar();
              if (bp < buflen && ch == 'a') {
                scanCommentChar();
                if (bp < buflen && ch == 't') {
                  scanCommentChar();
                  if (bp < buflen && ch == 'e') {
                    scanCommentChar();
                    if (bp < buflen && ch == 'd') {
                      deprecatedPrefix = true;
                      scanCommentChar();
                    }}}}}}}}}}}
```



| Language  | Tool       | Comment                                                      |
| --------- | ---------- | ------------------------------------------------------------ |
| PHP       | re2c       |                                                              |
| ninja     | re2c       |                                                              |
| yasm      | re2c       |                                                              |
| GCC < 3.0 | flex       |                                                              |
| Go??      | Hand-coded | Uses bison                                                   |
| perl5     |            |                                                              |
| Ruby      | Racc?      | Lexer/parser generator, also uses bison. Lexer seems to be generated but uses a custom lexer runtime? |

[^lexperf]:See [Baojian Hua](http://staff.ustc.edu.cn/~bjhua/) or [Martin v. Löwis](https://stackoverflow.com/questions/1101267/where-does-the-compiler-spend-most-of-its-time-during-parsing).
[^lexterms]: The terms scanning, lexing, and tokenizing are synonymous. Nonetheless, [Hotspot](https://github.com/AdoptOpenJDK/openjdk-jdk8u/blob/aa318070b27849f1fe00d14684b2a40f7b29bf79/langtools/src/share/classes/com/sun/tools/javac/parser/JavaTokenizer.java) uses the phrases lexer, scanner, and tokenizer simultaneously to refer to different translation units.
[^amdahl]: This phenomenon is called [Amdahl's law](https://en.wikipedia.org/wiki/Amdahl%27s_law). Mathematically, we have $\tilde{S}=\frac{1}{(1-p) + \frac{p}{s}}$, where $s$ is the speedup of the subtask, $p$ is the percent of the entire task the subtask represents, and $\tilde{S}$ is the speedup of the entire task. So if we could make the subtask “infinitely fast,” then we’d have $\lim _{s\to \infty }\tilde{S}=\frac {1}{1-p}$, which might be very small.