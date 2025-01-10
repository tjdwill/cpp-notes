# [CMake Generator Expressions](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#generator-expression-reference)

**Date**: 10 January 2025

In CMake, there are two phases to be aware of. The first phase, *configuration phase*, is where
CMake processes the CMakeLists.txt file, building an internal representation of the project. The
*generation phase* is the second phase of the project build. This is the phase where the build files
for the selected build system are generated. The problem with the configuration phase is that
certain properties and attributes are unknown at configure time when using a multi-generator system
like Visual Studio or XCode. In these systems, the generator isn't selected until the generation
phase, so properties such as `CMAKE_BUILD_TYPE` will have incorrect infromation if queried during
configuration time.

To combat this issue, CMake provides the *generator expression*, a syntax thatenables the user to
both query and conditionally set property values (and other values) during the generation phase.

## Basics

A generator expression, contrary to what I initially thought, does not generate anything. I thought
it owuld be something like a list comprehension in Python. Not so. It is simply a way to get
information during **generation**. It has the following form:

```cmake
$<...> # This is a generator expression
```

What is written for `...` varies. For example, CMake provides the `$<CONFIG>` expression which
returns the configuration name (*i.e.* build type).

### Conditional Generator Expressions

The book and documentation appear to emphasize the boolean form of
the generator expression. This form allows the user to conditionally return values upon the presence
of some condition. It is of form:

```cmake
# Boolean Generator Expressions

$<1:val>  # Returns "val"
$<0:val>  # Returns ""
```

Unlike the rest of CMake, enerator expressions are strict in that they only accept `1` or `0` for
true and false, respectively. To convert a CMake value to one of these values, we use the provided
`$<BOOL:...>` which will return either of the two accepted values.

One example of such a conditional generator expression is the one that queries the build type:

```cmake
$<CONFIG:Debug>  # 1 if true; 0 if false
```

Using this, we can build an expression that conditionally sets some property:

```cmake

set(compFlags <some flags here>)
# Assume some target MyTarget exists
target_compile_options(MyTarget PUBLIC
    $<$<CONFIG:Debug>:${compFlags}>
)
```

## Where Do I Use Generator Expressions?

As it appears to be with most of CMake, there is no set answer to this question. I think the answer
for *me* is: "where it makes sense." When a value is better suited for the generation phase, it
should be interacted with via a generator expression if possible. Try to be tasteful. I'm going to
get it wrong at times, but as I gain experience with CMake, the "feeling" should develop. 

CMake is more assertive regarding when *not* to use generator expressions. For one, only a subset of
commands support them. It is up to the user to review the documentation of the commands he uses to
ascertain if generator expressions are supported. For example, `if()` constructions do **not**
support them. Generator expressions are processed during generation, but `if()` is processed during
configuration, leading to an incompatibility.

I think, the best way to develop a sense of tsate and comfort is to use them and read the CMakeLists
of more experienced developers. At the end of the day, however, I'm going to do what I feel is best
(assuming there is no established standard practice for the given work).