# Large Scale C++ Design

Notes from John Lakos' book *Large Scale C++ Software Design* (1996), I found this book
when searching for resources for actually designing, implementing, and maintaining software
projects. After watching one of Lakos' talks, I was convinced that I could learn *a lot*
from his work. He's the first person I've seen talk about such design principles,
especially with such precision.

## Two Major Design Rules

1. No cyclic dependencies (physical or logical)
2. No long-distance friendships (friendships across components)

## Ch.2 Ground Rules

**Rule**: *Keep class data members private*

- First, it encourages the creation of abstractions
- Separates interface and implementation

Think of a class as a unique world. If an entity has direct access to the class' internal data, said entity becomes coupled to the class. Changes in the class measn the entity must change. Instead, try to keep your interface as a separate part of the class (don't expose private data in the interface).Yyou then have room to modify implementation as desired w/o affecting users. This rule, in a word, describes *encapsulation*. When data is not private, prefer the use of the `struct` keyword over `class`.

**Rule**: *Avoid data with external linkage at file (global) scope*.

Exposing data globally risks naming collisions. Instead:

- Put all global variables in a struct.
- Make said variables private and add `static` access functions.
    - Question: why not use a `namespace`? I don't think the namespace feature was robust in 1996.

**Rule**: *Avoid free functions (except operator functions) at file scope in header files;
avoid free functions w/ external linkage (__including__ operator functions) in
implementation files (.c, .cpp, .cxx, etc)*

Again, this is to avoid name collisions. Recall:

- *internal linkage*: the named entity is local to its translation unit and cannot collide
  w/ an identical name outside the translation unit @ link time.
  - Not visible to other translation units (a *translation unit* is the resulting
    intermediate file when all include directives are processed).
  - `static` keyword forces linkage to be internal for namespace-scoped entities (i.e. vars).
  - `const` variables and methods
  - enums in impl. files have internal linkage. Classes and inline functions as well.
- *external linkage*: the name is accessible to other TUs at link time.
    - non-inline member functions (including static members)
    - non-inline, non-static free functions

Don't place a definition w/ external linkage in a header file. Including the header in
multiple files produces MULTIPLY DEFINED SYMBOL.

Solution: 
    - Place into a struct with static member functions.
    - Free operator functions can't be nested w/in a class, so they remain file scoped.

**Rule**: *Avoid classic enumerations, typedefs, and constants @ file scope*

- Wrap into classes as static data members (for constants)
- Use class enums

**Rule**: *Except for include guards, avoid preprocessor macros in header files.*
- Macros create debugging headaches and aren't scopable. They're available to every TU that
  imports them, even unwittingly.

**Rule**: *Only classes, structures, unions, and free
operator functinos should be __declared__ at file scope in
a header file; only classes, structs, unions, and inline
(member or free operator) functions should be __defined__
at files scope in a header file.*

## Components

The *component* is the fundamental unit of design. Lakos presents two ideas: that of
*logical design* and *physical design*. Logical design consists of the programmatic
organization and structure of the project; class hierarchies, function hierarchies,
programming paradigm, etc. all fall under this design umbrella. Physical design, on the
other hand, is about how the proect is organized physically in terms of actual files. These
two ideas are not orthogonal; decisions in one aspect can and will affect the other.

Mr. John chooses the component as the atomic unit for multiple reasons, but the ones that
especially caught my attention were:

1. A single component can represent a complete abstraction.
2. When properly designed, a component can be reused without having to rewrite code (even
   via copy-paste).

A component consists of an interface and an implementation. In C++, a component consists of
exactly one header file (`.h`, `.hpp`) and (generally) exactly one implementation file
(`.c`, `.cpp`, `.cxx`, etc.). It represents a complete abstraction. Lakos then
distinguishes two types of interfaces.

A *logical* interface is all of the entities in a component that are programmatically reachable by the client.
The *physical* interface of a component is everything in the component's header (including
documentation comments). It holds information needed in order to use the component properly.

**Major**: 
