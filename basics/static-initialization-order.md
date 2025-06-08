# Static Initialization Order

Date: 5 June 2025

Recently, I ran into a problem that I want to take the time to write about. I
had a static map of values defined in a component, `foo`, and I attempted to use
this map to initialize another static map variable in component `bar`. 

```cpp
// foo.h
#ifndef INCLUDED_FOO
#define INCLUDED_FOO

#include <map>
#include <string>

namespace statics {
struct Foo {
	// some static variables
	const static std::map<int, std::string> kNameFromId; 
	const static std::map<std::string, int> kIdFromName; 
};
}
#endif // include guard
//------------------------------------------------------------------------------
// foo.cpp

#include "foo.h"

namespace statics {
	
const std::map<int, std::string> Foo::kNameFromId {
	{0, "tjdwill"},
	{1, "notTjdwill"},
};

// This is fine. kNameFromId is defined above in the same TU.
const std::map<std::string, int> Foo::kIdFromName {
	{kNameFromId.at(0), 0},
	{kNameFromId.at(1), 1},
};
}
//------------------------------------------------------------------------------
// bar.h
// ...
//------------------------------------------------------------------------------
// bar.cpp

#include "bar.h"

#include <foo.h>

#include <map>
#include <string>


namespace // unnamed namespace
{
// BAD. This can result in the Static Initialization Order Fiasco (SIOF)
const std::map<std::string, double> kDoubleFromName
{
	{statics::Foo::kNameFromId.at(0), 0.0},
	{statics::Foo::kNameFromId.at(1), 1.0},
};
}

int main()
{
	// ERROR. SIOF
	std::cout << kDoubleFromName.at(statics::Foo::kNameFromId.at(0)) << "\n";
}
```

When executing the program, I received the following error:

> **Exception thrown: read access violation. _Scary->_Myhead was nullptr**

(I was using Visual Studio 2022 in case that matters). As it turns out,
[someone else experienced the very same issue](https://developercommunity.visualstudio.com/t/Exception-thrown:-read-access-violation/10666202?sort=newest&viewtype=solutions),
and their solution was to inline `Foo::kNames`. Spoiler alert: it worked.
However, I wanted to understand *why* this worked, so that led me down a
research plunge.
	
## Variables

First, let's review variables for a bit. I thought I understood variables, but
thinking about them requires more precision in C++. For instance, I mistakenly
believed that global variables are only those variables defined at file scope.
This is not the case. More accurately, any variable that is declared/defined
outside of a function is a global variable. Meaning, a global variable lives in
*namespace* scope (including the global namespace, a.k.a file scope).

This is important because **global variables have
[static storage duration](https://en.cppreference.com/w/cpp/language/storage_duration.html)**;
they live for as long as the program executes (and are initialized before
`main()` executes). These two facts will be important later.

## `inline` specifier

What is the `inline` specifier? Originally, it was a directive telling the
compiler to insert the definition of an `inline`-specified function at any call
site of said function, aiding performance by precluding function call overhead. 

However, in the modern era, the compiler basically inlines as it sees fit,
regardless of whether the specifier is used or not. Instead, `inline` is now
used for the implication of the original use: an `inline` entity with external
linkage is able to be defined multiple times. In other words, `inline`d entities
are exempt from the One Definition Rule (ODR). The caveat is that all
definitions must be identical. 

Typically, developers declare and define an `inline` function or variable in a
header. Each TU that includes that header will effectively redefine that inlined
entity, but the definition is guaranteed to be the same. I want to re-emphasize
this point: including a header that has an inlined entity *redefines* that
entity at the inclusion site.

## Static Storage Initialization

According to
[cppreference.com](https://en.cppreference.com/w/cpp/language/initialization.html),
unless deferred, "all non-local variables with static storage duration are
initialized as part of program setup, before execution of the main function
begins." Entities with static storage duration are either initialized at compile
time (constant initialization), or, if needed, zero-initialized to then be set
at runtime via dynamic initialization.

After static initialization, *dynamic initialization* occurs. There are three
situations in which this type of initialization occurs in non-local variables,
but we are only concerned with the latter two:

> 2) *Partially-ordered dynamic initialization*, which applies to all inline
>    variables that are not an implicitly or explicitly instantiated
>    specialization. If a partially-ordered V is defined before ordered or
>    partially-ordered W in every translation unit, the initialization of V is
>    sequenced before the initialization of W (or happens-before, if the program
>    starts a thread).
> 3) *Ordered dynamic initialization*, which applies to all other non-local
>    variables: within a single translation unit, initialization of these
>    variables is always sequenced in exact order their definitions appear in
>    the source code. Initialization of static variables in different
>    translation units is indeterminately sequenced. Initialization of
>    thread-local variables in different translation units is unsequenced.

There are two pieces of knowledge here that will be useful. If all definitions
of an `inline` variable V occur before the definition of some ordered
dynamically-initialized variable W, V will be initialized before W.  

Secondly, if some ordered non-local variable X is defined before some other
ordered non-local Y *within the same translation unit*, X is initialized before
Y. Initialization of thread-local units in *different translation units is
**unspecified***, meaning there is no guarantee of initialization order. 

That latter observation is why the program above has a 50% chance of failure.
`kDoubleFromName` lives in a different translation unit (`.cpp` file/component)
than `Foo::kNameFromId`. As a result, it's possible for the former to be
initialized before the latter, resulting in a crash because its definition would
then be accessing an object that doesn't exist. If `Foo:kNameFromId` happens to
be initialized first, no error occurs. This situation is known as the *Static
Initialization Order Fiasco* (SIOF), and it's especially tricky because it only
has a chance of occurring. 

## Bringing All Together

With all of the information above, let's return to the code sample. In `foo`, a
static data member is defined. `bar`, a separate component/translation unit,
defines a global variable that references the `Foo::kNameFromId` static data
member. Both the static data member and the global variable are non-local
variables with static storage duration, meaning they are to be initialized
before `main` executes. 

However, because they live in separate TUs, there is no specified order in which
these two variables, `Foo::kNameFromId` and `kDoubleFromName`, are to be
initialized. Therefore, there is a chance that `kDoubleFromName` is initialized
before `Foo::kNameFromId`. This results in a crash because the definition of
`kDoubleFromName` references `Foo::kNameFromId`, an entity that doesn't yet
exist in this context.

The de-facto method of solving this problem is to use the [*Construct on First
Use*](https://isocpp.org/wiki/faq/ctors#static-init-order-on-first-use) idiom.
Essentially, we would define that same `kDoubleFromName` map object as a static
variable within a function. The variable will be initialized on the first call
to the function, which is *guaranteed* to happen *after* `main()` begins its
execution. As a result, `Foo::kNameFromId` will be initialized by the time
`kDoubleFromName` (now a static variable in a function) is initialized. 

```cpp
// bar.cpp
// IMPROVED VERSION
#include "bar.h"

#include <foo.h>

#include <map>
#include <string>


namespace // unnamed namespace
{
// Use a function with a static object.
// The object won't be instantiated until the first time this function is
// called, so it is guaranteed to be instantiated *after* the static global map.
auto doubleFromName() -> std::map<std::string, double> const&
{
	static const std::map<std::string, double> doublesMap
	{
		{statics::Foo::kNameFromId.at(0), 0.0},
		{statics::Foo::kNameFromId.at(1), 1.0},
	};
	return doublesMap;
}
}

int main()
{
	// This will always work since `doubleFromName` is called during `main`'s 
	// execution. The inner static variable is guaranteed to be initialized
	// after the global variables and static data members.
	std::cout << doubleFromName.at(statics::Foo::kNameFromId.at(0)) << "\n";
}
```

### Why Did The `inline` Solution Work?

As noted previously, inlining `Foo:kNameFromId` also works as a solution:

```cpp
// foo.h
#ifndef INCLUDED_FOO
#define INCLUDED_FOO

#include <map>
#include <string>

namespace statics {
struct Foo {
	// some static variables
	static const std::map<int, std::string> kNameFromId; 
	static const std::map<std::string, int> kIdFromName; 
};

inline const std::map<int, std::string> Foo::kNameFromId {
	{0, "tjdwill"},
	{1, "notTjdwill"},
};

// This is fine. kNameFromId is defined above in the same TU.
inline const std::map<std::string, int> Foo::kIdFromName {
	{kNameFromId.at(0), 0},
	{kNameFromId.at(1), 1},
};
}
#endif // include guard
//------------------------------------------------------------------------------
// foo.cpp

#include "foo.h"
//------------------------------------------------------------------------------
// bar.h
// ...
//------------------------------------------------------------------------------
// bar.cpp

#include "bar.h"

#include <foo.h>

#include <map>
#include <string>


namespace // unnamed namespace
{
// This WORKS because Foo::kNameFromId is inlined.
const std::map<std::string, double> kDoubleFromName
{
	{statics::Foo::kNameFromId.at(0), 0.0},
	{statics::Foo::kNameFromId.at(1), 1.0},
};
}

int main()
{
	std::cout << kDoubleFromName.at(statics::Foo::kNameFromId.at(0)) << "\n";
}
```

Remember, when we include the header that defines some `inline` entity, that
entity is *redefined* in the translation unit including the header. So, `bar`
now has its own copy of that definition `Foo::kNameFromId` that lives in the
same TU as `kDoubleFromName`. Additionally, based on the rules of static
initialization, the data member undergoes partially-dynamic initialization
because it's inlined. Since the `#include` directive that defines
`Foo::kNameFromId` appears *before* the definition of `kDoubleFromName`,
`Foo::kNameFromId` will be initialized before `kDoubleFromName` in accordance
with the associated rules. In other words, the program is no longer susceptible
to SIOF, making `inline` a potential solution. 

There is a discussion to be had about which solution is better architecturally,
but my primary concern was understanding *why* the solutions work. I know have a
better understanding than I did before.

## References

1. Pablo Arais: [*C++ - Initialization of Static Variables*](https://pabloariasal.github.io/2020/01/02/static-variable-initialization/)
0. Pablo Arias: [*C++ - Inline Variables and Functions*](https://pabloariasal.github.io/2019/02/28/cpp-inlining/)
0. cppreference: [*Static Initialization Order Fiasco*](https://en.cppreference.com/w/cpp/language/siof.html)
0. cppreference: [*Initialization*](https://en.cppreference.com/w/cpp/language/initialization.html#Non-local_variables)
0. cppreference: [*`inline` specifier*](https://en.cppreference.com/w/cpp/language/inline.html)
0. ISOCPP: [*Constructors*](https://isocpp.org/wiki/faq/ctors#static-init-order)
0. Learn C++: [*Inline functions and variables*](https://www.learncpp.com/cpp-tutorial/inline-functions-and-variables/)