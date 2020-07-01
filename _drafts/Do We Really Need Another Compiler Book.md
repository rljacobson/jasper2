# Do We Really Need Another Compiler Book?

*Actually, we need at least three, probably more.* *I explain why all compiler books are the same and what I think we should do about it.*

## The Genesis of the Compiler Fetish

Since the dawn of programmable computers, programmers and computer scientists have nearly universally had a compiler fetish. [Grace Hopper](https://en.wikipedia.org/wiki/Grace_Hopper) was the first to implement one. It was in 1952 for a language she appropriately called [A-0](http://xover.mud.at/~marty/iug2/p243-hopper.pdf). Some argue—incorrectly in my view—that A-0 functions more as a linker-loader and thus that the first compiler in the modern sense of the word was [FORTRAN](https://archive.computerhistory.org/resources/text/Fortran/102663113.05.01.acc.pdf), a language which has been in continuous use to the present day. No matter how one characterizes their differences, the first FORTRAN, completed in 1957, was fundamentally more of a compiler in the modern sense than A-0. Produced by IBM, it was also the first commercial compiler and took a team of 13 engineers led by John Backus two real years and 18 man-years to complete. The second oldest language in continuous use is Lisp, for which was written the very first self-hosting compiler in 1962. Very few other languages from the '50s and ‘60s survive to this day.

But the mathematical theory of programming languages flourished during this era, while dozens of papers describing practical programming languages and their implementations were published. Over time it became possible to talk about the *theory of compiler construction* as a thing: ad hoc implementations gave way to formalized techniques, and certain algorithms and designs emerged as standard tools in the field. Once something is a *thing*, books can be written about that thing.

## Siren Land, or The Landscape of Compiler Texts

Nearly all compiler texts fall into one of a small number of genres:

1. academic surveys of algorithms, techniques, and designs; the stereotypical *textbook*
2. “Let’s build a toy compiler”
3. mathematical texts focused narrowly on a single aspect of compiler construction
4. practical tool-oriented manuals

### The Textbook

The history of the theory of formal languages and compiler construction in academia has defined the content and format of virtually all compiler construction textbooks. They all include the same topics, no matter how outdated, and exclude the same material, no matter how central to compiler construction as actually practiced. This phenomenon is tragically common within academia. Mathematician Underwood Dudley explores it with great wit in [his book review](https://doi.org/10.1080/00029890.1988.11972109) of *Calculus with Analytic Geometry* by George F. Simmons:

> In the 85 calculus books I examined, almost all of them had the Norman window problem—the rectangle surmounted by a semicircle, fixed perimeter, maximize the area. The semicircle always "surmounts." This is the sole surviving use of "surmounted" in the English language, except for the silo, a cylinder surmounted by a hemisphere. Only one author had the courage to say that the window was a semicircle on top of a rectangle. All the books had the box made by cutting the corners out of a flat sheet, all have the ladder sliding down the wall, all had the conical tank with changing height of water, all had the tin can with fixed surface area and maximum volume, all had the V-shaped trough, all had the field to fence, with or without a river flowing (in a dead straight line) along one side, all had the wire—usually wire, but sometimes string—cut into two pieces to be formed into a circle and a square, though some daring authors made circles and equilateral triangles. There are only finitely many calculus problems, and their number is *very* finite.
>
> ... There are no conical reservoirs outside of calculus books. Real reservoirs are cylindrical, or perhaps rectangular. The reason for this is found in the texts: in the problems, the conical reservoirs usually have a leak at the bottom.

Why do we see so many similar compiler textbooks? No doubt some of the same reasons Dudley gives in his book review apply to compiler textbooks as well, and I encourage the interested reader to consult Dudley for those details. But compiler construction is not freshman calculus. We need to look further for the whole picture.

