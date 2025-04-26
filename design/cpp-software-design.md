Notes on *C++ Software Design* by Klaus Iglberger

Chapter 1:
------------------

## Guideline 1: Understand the Importance of Software Design

Though the focus tends to be on them, language features are not the most important, critical aspect
of software design. Since we want software that fits many characteristics— extensible, easily
changed, testable, modular, etc.—we must pay special attention to the *structure* of our project.
Part of this task is curating dependencies, managing how the components of our software relate to
one another and the abstractions we rely upon as we build our product.  To put it explicitly:

> Software design \[is\] the art of managing dependencies and abstractions. (*pg. 27*)

Klaus provides three levels of software development in this portion of the book: **Software
Architecture**, **Software Design**, and **Implementation Details**. Though he tags software design
as the level in which Lakosian principles apply (physical and logical relationships -> components),
I think the three levels are Lakosian on a spectrum from most physically-oriented to most
logically-oriented. Architectural decisions such as how the product is packaged, distributed,
structured, etc. all affect the physical structure of the product—the way the work is *physically*
represented and stored within electronics. On the other end, implementation details are more
logically-oriented. Whether we use *RAII* or not does not (should not) affect how the project is
structured electronically, but it does affect how the software is programmed, the techniques and
logical entities used.

## Guideline 2: Design for Change

As Shakespeare once put it: "A rose by any other name would smell as sweet". Software engineers
prove this true with the *Single Responsibility Principle*. It goes by many names (Klaus uses
"Separation of Concerns"), but the underlying principle is the same: programmatic entities with
focused purpose tend to make a codebase that is easier and more pleasant to change. Entities (ex.
classes) that take on multiple roles lead to a messier, more coupled design. For example, imagine
how hairy a plotting library would be if the plot function were also responsible for producing the
data it calculates (really imagine it. What happens if it needs to solve differential equations or
integrals?). Now wonder: what happens if we need to add another equation type? An additional
variable axis?