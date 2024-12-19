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
- [Levelization](#levelization)
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

Honestly, there's not much to write for this one. The main point of the chapter is about
how physical hierarchy makes testing easier and how it enables distributed testing instead
of one large test at the top hierarchy. Lakos also introduces a metric, *cumulative
component dependency* (CCD) to quantify physical design. 

**TODO**: Expound on topics when necessary.

## Levelization

Goal: Remove and preempt cyclic dependencies via levelization techniques.

This is the first chapter where we get into techniques for reducing cyclic dependencies.
Lakos establishes that cyclic dependencies can arise in previously acyclic codebases due to
increased feature requests (and associated time constraints). A design decision that may appear
"convenient" could result in severe buyer's remorse down the line. 

> **Principle:** *Allowing two components to "know" about each other via `#include`
> directives implies cyclic dependency.* 

### Escalation

The first technique J.L. discusses is *escalation*. This is the practice of taking
interdependent functionality of cyclically-dependent components and elevating it to a
higher level, potentially creating a new component in the process. For example, given two
classes `Foo` and `Bar` that can mutually convert between each other, we can introduce a new
class `FooBarConverter` in a higher-level component that performs the conversion instead. The difference is, 
`FooBarConverter` now depends on `Foo` and `Bar` components, and, after removing the
relevant functions from the two original classes, `Foo` and `Bar` become mutually
independent.

### Demotion

The analogue to escalation, *demotion* is the practice of extracting common functionality
between (cyclic) components and placing them in a lower level component, potentially
creating the component in the process. The example J.L. gives in the text is that of a
common set of useful functions being extracted to a utility struct and placed lower in the
hierarchy (`GeomUtilCore`). 

### Opaque Pointers

Typically, when we think of using a type `T`, that type is used *in size*; the compiler
needs to know `T`'s size and memory layout in order to compile the program successfully. 
However, we can also use a type *in name only*. In this sense, we only store a pointer or
reference to the type, never dereferencing the pointer or accessing the type's members or
methods (all of which require the definition to be known).

> **Definition**: A function (or component) `f` uses a type `T` *in size* if compiling the
> body of `f` requires having first seen the definition of `f`. The entity uses the type
> *in name only* if it can compile **without** needing to see the definition of `T`.

- Using a type in size introduces a compile-time dependency on the component that defines
  it.
- Using a type in name only implies **no physical dependency**, even at link time.
- If a component or function depends on another entity that *does* use `T` in size, it will
  physically depend on the component defining `T` via transience.
- *Opaque Pointer*: a pointer to a type whose definition is not included in the pointer's
  translation unit.

The idea here is that a class can store a pointer to some type without needed to know what
the pointer points to. More explicitly, we can store memory addresses with no dependency so
long as we don't attempt to interact with what's stored at that address. How is this
useful for levelization?

The example Lakos uses is that of a container whose containees store a pointer to the
container. The container needs to know the definition of the containee type, so there's a
dependency there. A cyclic dependency is introduced when the containee uses the pointer to
the container substantively. One example of which is using the pointer to call a method of
the container during the implementation of some containee method.

To remove this cyclic dependency, Lakos recommends making the pointer opaque: 

0. Remove the uses of the pointer, only allowing the pointer to be retrieved (and possibly
   set).
1. Escalate the removed functionality to the container itself, making it a static method
   that takes the containee by reference (or pointer). Then, you can simply call the
   container's method as done before.

    ```cpp
    /* INSTEAD OF */
    // containee.cpp
    //...
    auto Containee::some_query() const -> int
    {
        return my_container()->get_val();  // requires the definition of the container
    }

    /* DO */
    // container.hpp
    class Containee;
    class Container
    {
        //...
        static auto get_val(Containee const& item) -> int;
    };

    // container.cpp
    #include "container.h"
    #include "containee.h"

    auto Container::get_val(Containee const& item) -> int
    {
        return item.pointer()->get_val();
    }
    ```

### Dumb Data

> **Dumb Data**: Any kind of information that an object holds but does not know how to
> interpret.

This is the generalized concept of which the opaque pointer is an instance. The idea behind
dumb data is to include data within a lower-level class that can only be interpreted by
some entity in an upper-level class. the lower class doesn't interact or use the data in
any way other than simple data retrieval operations. The upper class uses this data to
implement some functionality. 

To me, dumb data as used in the book is the practice of using simple data types (ex.
`int`) to represent some information that can then be used by a higher-level entity that
interprets that information. It's a form of encoding to obviate the need to introduce
cyclic logical dependencies and/or physical dependencies.

One problem with this method is that the programmer is reponsible for keeping track of what
a given dummy value means. 

> **Principle**: Dumb data can be used to break *in-name-only* dependencies, facilitate
> testability, and reduce implementation size. However, opaque pointers can preserve both
> type safety and encapsulation; dumb data, in general, cannot.

### Redundancy

Sometimes, instead of adding a dependency via reuse, it may be more effective to simply
repeat code in different components. When the cost of coupling outweighs the extent to
which you use the reused component, consider whether or not it is actually worthwhile to
add such a dependency.

---

## Questions and Thoughts

0. How do we begin to design a system? Obviously, we are unlikely to get it right the first
   time, but what are some common steps that occur when successfully designing a system,
   large or small?
1. What's the best include guard naming schema that minimizes collision and enables easy
   modification?


