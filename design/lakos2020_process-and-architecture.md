# Large Scale C++ Design

Volume 1: Process and Architecture

## Terminology

- *Component*: The smallest atomic unit of physical design (`.h`/`.cpp` pair in
  C++)  
- *External linkage*: The idea that a logical entity is accessible outside of
	its defining translation unit.  
	- *Linkage*, as Lakos puts it, is "used to determine whether a given
	declaration or definition refer to the same logcial entity" (pg. 159)  


## The Four Fundamental Properties of a Component



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
