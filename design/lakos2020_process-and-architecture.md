# Large Scale C++ Design

Volume 1: Process and Architecture

## Terminology

- *Component*: The smallest atomic unit of physical design (`.h`/`.cpp` pair in
  C++)  
- *Linkage*, as Lakos puts it, is "used to determine whether a given declaration
  or definition refer to the same logical entity" (pg. 159)  
	- *External linkage*: The idea that a logical entity is accessible (even
	definable) outside of the translation unit in which it's declared.  
	- *Internal linkage*: The logical entity denoted by a declaration can be
	defined in a different scope but not a different translation unit.  
		- By default, the following have internal linkage:  
			- `const` objects (unless declared `extern`)  
			- `constexpr` objects  
			- `typedef` objects (what about `using` type aliases?)  
			- `static` objects in namespace scope  
			- `static` functions  
			- Anything in an unnamed namespace scope  
	- *No linkage*: The entity must be defined in its declaration scope and
	translation unit.
- *Bindage*: Essentially, whether the compiler is responsible for resolving the
  definition (internal bindage), the linker (external), or either (dual).

## The Four Fundamental Properties of a Component

### Component Property 1

>The `.cpp` file incorporates its corresponding `.h` file as the first
>substantive line of code.  

"Substantive" meaning "non-comment" . The primary reason Lakos suggests this is
that every header will "compile in isolation," thereby removing an entire source
of bugs from include order dependency (pg. 210). 

To elaborate on the idea of the header compiling in isolation, consider how
includes work. Upon reading a given include directive, the preprocessor opens
that file and pastes its contents directly at the inclusion site, transforming
the source file. Sometime later, the compiler begins processing the
transformed/expanded source file. If the header is the first substantive line of
code, its code is processed first, so any uses of undeclared symbols will be
caught as a compilation error, signaling that the developer did not properly
define the given header.

The example Lakos uses is that of `std::ostream&`. If we have some `operator<<`
overload declaration in a header:

```cpp
// foo.h
// component_property_1
#ifndef INCLUDED_FOO
#define INCLUDED_FOO

// No declaration of std::ostream!

class Foo
{
public:
	Foo();
};

auto operator<<(Foo const& object) -> std::ostream&;

#endif // include guard
```

If we have some `.cpp` that includes `<ostream>` **before** `foo.h`, the
compilation will still succeed because a declaration of `std::ostream` is found
in `<ostream>` . However, if some `random_entity.cpp` includes `<foo.h>` alone,
for example, the compilation will fail because `std::ostream` is never declared
before its use. This is a subtle, potentially nasty bug that can take a long
time to troubleshoot. Component property 1 precludes this scenario entirely.

### Component Property 2

> Logical constructs having external linkage defined in a `.cpp` file — and not
> otherwise effectively rendered externally invisible — are declared in the
> corresponding `.h` file.

The purpose of this property is to prevent violations of the one definition rule
(ODR). No component should export the definition of an externally-linked symbol
without also exporting its declaration.

### Component Property 3

> Logical constructs having external or dual bindage that are declared within
> the header file of a component, if defined at all, are defined within that
> component only

It makes little sense for a component X to depend on another component Y for the
definition of an entity X declares (in a non-forward declaring context).

### Component Property 4

> There are no local “forward” declarations for a logical construct having
> external or dual bindage defined (uniquely) by another component; instead, the
> `.h` file of that component is `#include`-ed to obtain the needed declaration

I think this is meant in the context of implementation files. Forward
declarations of classes are valid within a header.

## 2.5 Component Source-Code Organization

Here, Lakos presents a rigorous structure for a component. 

1. Unique identifier (a comment with the component name)  
2. Include guard  
3. Component-level documentation and usage examples  
4. `#include`s needed to compile the header  
	1. Headers for intra-package components are included first  
	2. Headers for intra-package-group (different package, same package group)
	are included next  
	3. Headers in other package groups  
	4. Standard headers  
		- Note that includes within each level are sorted alphabetically.  
5. Open the enterprise-wide namespace  
	- Create any relevant forward declarations (one per line) for classes
	outside of the current package  
6. Package-level namespace  
	 1. Forward declarations for classes within the package (one per line)  
7. Declare every piece of logical content the header provides.  
	- One or more classes (or a single utility `struct`)  
	- Each class is followed by declarations of any associated free operators  
	- *IF* we are compelled to have "aspect" functions (functions like `swap`
	that have "universally consistent syntax and semantics" [pg. 335]) their
	declarations follow the free operators.  
8. Source code of any definitions shared across translation units are placed
   outside of the lexical scope of the class (so, outside of the class
   definition) and below the entire public interface of the component.  
9. Logical interface and implementation follows public interface and is
   delimited by a prominent banner (comment)  
	- All inline member function and member function template definitions follow
	in the same relative declaration order  
	- Because free functions defined at namespace scope are self-declaring (see
	*1.3.1*), we define free operators and free aspect functions outside of the
	namespace in which they are declared, closing the package namespace and then
	package-qualifying each free operator def., followed by free aspect defs.  
10. Any template specializations for templates defined in other packages  
	- close the package namespace first  
11. Close the enterprise-wide namespace and terminate the (internal) include
    guard, ending the component header. 

(Slightly-modified) Example (pg. 337-340):

```cpp
// xyza_component.h -*-C++-*-

#ifndef INCLUDED_XYZA_COMPONENT
#define INCLUDED_XYZA_COMPONENT

//@PURPOSE: This is a one-line sentence.
//
//@CLASSES:
// xyza::Class1: one-line phrase
// xyza::Class2: one-line phrase
// ...
//
//@DESCRIPTION: This component ...
// ...
///Usage
///-----
// ...

/* Intra-package includes */
#include <xyza_component1.h>
#include <xyza_component2.h>
#include <xyza_component3.h>
// ...

/* Intra-package-group include */
#include <xyzb_component1.h>
#include <xyzb_component2.h>
#include <xyzc_component1.h>
#include <xyzc_component2.h>
#include <xyzc_component3.h>
#include <xyzd_component1.h>
// ...

/* External package group includes */
#include <qrsx_component1.h>
// ...
#include <qrsz_component3.h>
// ...
#include <zyxq_component3.h>

/* Standard includes */
#include <iosfwd>
// ...


namespace MyLongCompanyName {  /* Enterprise-wide namespace */

/* External-package forward declarations */
namespace abcz { class ClassA; }
namespace qrsy { class ClassA; }
namespace qrsy { class ClassB; }

namespace xyza {  /* package-level namespace */

// Local declarations of intra-package types
class ClassA;
class ClassB;

// Sequence of component classes
class Class1 {
	// ...
  public:
	Class1();
	// ...
	auto func1(qrsy::ClassA *object) -> int;
	// ...
	auto func2(qrsy::ClassB *object) -> int;
	
	// ...
};

auto operator==(const Class1& lhs, const Class1& rhs) -> bool;
// ...

auto operator<<(std::ostream& stream, const Class1& object) -> std::ostream&;
// ...

void swap(Class1& a, Class1& b);

class Class2 {
	// ...
};

// ...

// ====================================
//      INLINE FUNCTION DEFINITIONS
// ====================================
inline
auto Class1::func2(qrsy::ClassB *object) -> int
{
	// ...
}
// ...

} // close package namespace

/* Free operators and aspects */
inline
auto xyza::operator==(const Class1& lhs, const Class1& rhs) -> bool
{
	// ...
}

// ...

inline
void xyza::swap(Class1& a, Class1& b)
{
	// ...
} 

// ...

/* Template specializations */
namespace abcmf {
template <>
struct IsBig<xyza::Class1>
					: std::true_type {};
} // close namespace abcmf

// ...

} // close enterprise namespace
#endif
```

And the implementation:

```cpp
// xyza_component.cpp -*-C++-*-

// ...
// ... (implementation overview)
// ...

#include <xyza_component.h>
#include <xyzb_component4.h>
#include <xyzb_component5.h>
// ...
#include <qrsx_component2.h>
// ...
#include <iostream>
// ...

namespace MyLongCompanyName {
namespace xyza {

namespace {
// ...
} // close unnamed namespace

auto Class1::func1(qrsy::ClassA *object) -> int
{
	// ...
	// ...
}
// ...

} // close package namespace

auto xyza::operator<<(
	std::ostream& stream,
	const Class1& object) -> std::ostream&
{
	// ...
}
// ...

} // close enterprise namespace
```

## 2.6 Component Design Rules

Here are some (not all) of the design rules Lakos establishes:

### Design Imperatives

- > Cyclic physical dependencies among components are not permitted.  
- > Access to the private details of a logical construct must not span the
  > boundaries of the physical aggregate in which it is defined — e.g.,
  > “long-distance” (inter-component) friendship is not permitted.

### Design Rules

- > The `.h` file of each component must contain a unique and predictable
  > include guard: **INCLUDED**_PACKAGE_COMPONENTBASE (e.g.,
  > `INCLUDED_BDLT_DATETIME`).

- > Any direct substantive use of one component by another requires the
  > corresponding `#include` directive to be present directly in the client
  > component (in either the `.h` or `.cpp` file, but not both) unless indirect
  > inclusion is implied by intrinsic and substantial compile-time dependency —
  > i.e., public inheritance (nothing else).

### Guidelines

- > The `.h` file should contain only those `#include` directives (or, where
  > appropriate, local \[“forward”\] class declarations) necessary to ensure
  > that the header file com- piles in isolation.
- Reasons to include a header in another header (pg. 355):  
	1. **Is-A** relationship (inheritance)  
	2. **Has-A** relationship (composition; not Holds-A or
	Uses-In-The-Interface)  
	3. **Inline** (any object used substantively in an inline definition)  
	4. **Enum**  
	5. **Typedef** (to an explicit template specialization like `std::string` or
	``std::vector`)  