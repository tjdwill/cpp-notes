# Large Scale C++ Design

Notes from John Lakos' book *Large Scale C++ Software Design* (1996), I found this book
when searching for resources for actually designing, implementing, and maintaining software
projects. After watching one of Lakos' talks, I was convinced that I could learn *a lot*
from his work. He's the first person I've seen talk about such design principles,
especially with such precision.

Contents:

- [Ground Rules](#ground-rules)
- [Components](#components)
- [Physical Hierarchy](#physical-hierarchy)
- [Questions](#questions-and-thoughts)

---

## Lakos' Two Rules

1. No cyclic dependencies (physical or logical)
2. No long-distance friendships (friendships across components)

## Ground Rules

> **Rule**: *Keep class data members private*

- First, it encourages the creation of abstractions
- Separates interface and implementation

Think of a class as a unique world. If an entity has direct access to the class' internal
data, said entity becomes coupled to the class. Changes in the class measn the entity must
change. Instead, try to keep your interface as a separate part of the class (don't expose
private data in the interface). You then have room to modify implementation as desired w/o
affecting users. This rule, in a word, describes *encapsulation*. When data is not private,
prefer the use of the `struct` keyword over `class`.

> **Rule**: *Avoid data with external linkage at file (global) scope*.

Exposing data globally risks naming collisions. Instead:

- Put all global variables in a struct.
- Make said variables private and add `static` access functions.
    - Question: why not use a `namespace`? I don't think the namespace feature was robust in 1996.

> **Rule**: *Avoid free functions (except operator functions) at file scope in header files;
avoid free functions w/ external linkage (__including__ operator functions) in
implementation files (.c, .cpp, .cxx, etc)*

Again, this is to avoid name collisions. Recall:

- *__internal linkage__*: the named entity is local to its translation unit and cannot collide
  w/ an identical name outside the translation unit @ link time.
  - Not visible to other translation units (a *translation unit* is the resulting
    intermediate file when all include directives are processed).
  - `static` keyword forces linkage to be internal for namespace-scoped entities (i.e. vars).
  - `const` variables and methods
  - enums in impl. files have internal linkage. Classes and inline functions as well.
- *__external linkage__*: the name is accessible to other TUs at link time.
    - non-inline member functions (including static members)
    - non-inline, non-static free functions

Don't place a definition w/ external linkage in a header file. Including the header in
multiple files produces MULTIPLY DEFINED SYMBOL.

Solution: 

    - Place free functions in a struct as static member functions.
    - Free operator functions can't be nested w/in a class, so they remain file scoped.

> **Rule**: *Avoid classic enumerations, typedefs, and constants @ file scope*

- Wrap them in classes as static data members (for constants)
- Use class enums for less ambiguity.

> **Rule**: *Except for include guards, avoid preprocessor macros in header files.*
- Macros create debugging headaches and aren't scopable. They're available to every TU that
  imports them, even unwittingly.

> **Rule**: *Only classes, structures, unions, and free
operator functions should be __declared__ at file scope in a header file; only classes,
structs, unions, and inline (member or free operator) functions should be __defined__ at
files scope in a header file.*

## Components

The *__component__* is the fundamental unit of design. Lakos presents two ideas: that of
logical design and physical design. *__Logical design__* consists of the programmatic
organization and structure of the project; class hierarchies, function hierarchies,
programming paradigm, etc. all fall under this design umbrella. *__Physical design__*, on
the other hand, is about how the proect is organized physically in terms of actual files.
These two ideas are not orthogonal; decisions in one aspect can and will affect the other.

Mr. John chooses the component as the atomic unit for multiple reasons, but the ones that
especially caught my attention were:

1. A single component can represent a complete abstraction.
2. When properly designed, a component can be reused without having to rewrite code (even
   via copy-paste).

A component consists of an interface and an implementation. In C++, a component consists of
exactly one header file (`.h`, `.hpp`) and (generally) exactly one implementation file
(`.c`, `.cpp`, `.cxx`, etc.). It represents a complete abstraction. Lakos then
distinguishes two types of interfaces.

A component's *logical* interface is all of the entities in a component that are
programmatically reachable by the client.  The *physical* interface of a component is
everything in the component's header (including documentation comments). It holds
information needed in order to use the component properly.

> **Major**: *The first substantive line of code in _any_
component's implementation file should be the inclusion of the component's header.*

Rationale: Doing so ensures the header compiles in isolation, meaning all necessary
declarations are made. This way, we preclude inclusion order errors.

> **Major**: *Avoid defining externally-linked entitites in an implementation that aren't
> declared in the corresponding interface file.*

Rationale: We want our dependencies to be explicit.

> **Major**: *Avoid accessing a definition w/ external linkage in another component via
> local declaration.  Instead, include the header for that component.*

This rule and the previous one are basically referencing situations where someone forward
declares a free function or `extern`s a global variable. In addition to keeping dependecies
explicit, the recommended practice ensures logical interfaces remain up-to-date. Lakos uses
a function whose parameter list changes in the source but not in the client as an example
(p. 115-118).

> **Definition**: *A component y `DependsOn` a component x if x is needed to compile or
> link y.*

`DependsOn` relationships are transitive. If x depends on y, and y depends on z, then x
depends on z.

> **Principle**: *A component defining a function will __usually__ have a physical
> dependency on any component defining a type used by that function.*

- Logical relationships `IsA` (inheritance) and `HasA` (direct composition/layering)
  implies that a class will depend on the type at compile time. This is also true for any
  client of the class.

> **Principle**: *A component defining a class that IsA or HasA user-defined type
> __always__ has a compile-time dependency on the component defining that type.*

Other relationships `HoldsA` (the class embeds a pointer or reference to a type) and `Uses`
(the class has a member function that names the type) imply link-time dependencies across
components.

### A note on when to use `#include` directives

Per Section 6.3.7, we generally use include directives in a component header when:

1. An entity in the component establishes an `IsA` relationship.
2. An entity in the component establishes a `HasA` relationship.
3. An inline-declared function in the component header uses a class defined in the external
   component in size.
    - Ex. calling a method of the type.

In other words, if the definition of the class is needed in order to use it, make the
dependency explicit by including the defining component's header.


### Friendship

Recall: A `friend` in C++ is an entity that has access to a
class' internal representation (`private` and `protected` members).

There was a lot in this section, but the moral is to never use long-distance friendship.
*__Long-distance__* friendship is when a function is declared a `friend` across component
boundaries. 

- Friendship w/in a component maintains encapsulation (data hiding). 
- Friendship extends a class' logical interface.
- Use friendship judiciously; extensive use will cause maintainence headaches when the
  internal representation of the class changes.

## Physical Hierarchy

---

## Questions and Thoughts

0. How do we begin to design a system? Obviously, we are unlikely to get it right the first
time, but what are some common steps that occur when successfully designing a system, large
or small?
1. What's the best include guard naming schema that minimizes collision and enables easy
   modification?
