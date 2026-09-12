Date: 16 October 2025

One frustrating aspect of CMake is the sheer number of ways to achieve the same
desired result. Exacerbating this issue is that every developer seems to have a
different idea of what true "modern" CMake looks like. As we all twirl our
mustaches and adjust our monocles in pursuit of this Platonic ideal, here are
some notes from an exchange I had with an actual CMake developer (Redditor
/u/not-a-novel-account, A.K.A nickelpro) about includes (via FILE_SET).

First, let's talk about header ownership because this concept confused me
initially. A given target , a named collection of properties typically declared
by `add_[library|executable]()`, owns a given header file if that target
declares and defines entities in said header as part of the target's interface.
This really only makes sense if the target is a library, so let's assume that's
the case. Other targets can *use* that header (think of Rust's "borrow"
concept), but the header *belongs* to the owning target. To quote the helpful
CMake dev directly:

> If library Alpha has a function `foo()`, then the header with the declaration
> for `foo()` belongs to Alpha. If library Bravo has a function `bar()`, the
> header declaring `bar()` belongs to Bravo. The header for `bar()`, say
> `bar.hpp`, should not appear in the `target_sources()` command for library
> Alpha.

If the header is part of the library's interface, it should be associated with
that library target via `target_sources()`.  We define the list of headers owned
by a target via the `FILE_SET` portion of `target_sources()`. This directive
allows developers to specify both *which* headers are owned by a target as well
as *where* to find them (see `BASE_DIRS` and `FILES`). Files in a `PRIVATE` set
are needed to build the given target; `INTERFACE` files are required to consume
the target. `PUBLIC` files are for both. By defining the FILE_SET for a target,
dependents can inherit the include paths associated with the target via
`target_link_libraries()`.

So, each target is responsible for its own includes. If a given target X wishes
to interface with another target Y's includes, X must link to the Y via
`target_link_libraries()`. This will allow X to inherit Y's include paths
without having to specify them in `target_sources()`. Based on this paradigm, it
would appear that the preferred way of consuming third party entities is to:

0. (Build and) Install the given package via CMake (look into
   `target_include_directories()` and other alternatives for non-CMake
   projects). 
1. Import the package in the relevant CMakeLists.txt (CML) via `find_package()`.
2. Link our desired target to the relevant imported package target. - One may
	need to read the third party package's CMake in order to ascertain the
	correct naming conventions to use for the targets.
	
## Determining Your Project's Include Directive Format

If familiar with the [angle-bracketed include
directive](https://gcc.gnu.org/onlinedocs/cpp/Include-Syntax.html) form, you've
likely used libraries that have their names prepended to the include directive.
For example, the Boost library has includes like:

```cpp
#include <boost/numeric/conversion/cast.hpp>
```

CMake's FILE_SET feature enables developers to set their library component's
include directive based on the `BASE_DIRS` parameter. For FILE_SETS of the
HEADERS type, the directories listed in `BASE_DIRS` will be added to the
target's header search path. If the FILE_SET is INTERFACE or PUBLIC, this
applies to consumers as well. As a result, we can choose the include directive
format for our library by choosing which directories are added to BASE_DIRS
(note: see the interplay between FILE_SET and `install()` to understand why this
works for files listed relative to something like `CMAKE_CURRENT_BINARY_DIR`).

For example, say we have a library, mbl (My Basic Library), that has multiple
subdirectories:

```
mcl/
|
├── core/
|	├── someComponent.cpp
|	├── someComponent.h
|	├── ...
|	└── CMakeLists.txt
├── ...
└── CMakeLists.txt
```


Assume that we build out-of-source such that the build directory is outside of
the entire project tree. To have an include of format `#include <mcl/core/someComponent.hpp>`,  
the FILE_SET would look like the following:

```cmake
# mcl/core/CMakeLists.txt

# Generated headers should be placed in ${CMAKE_CURRENT_BINARY_DIR}/${inclusion_prefix}.
# This way, adding ${CMAKE_CURRENT_BINARY_DIR} to BASE_DIRS
# will enable the `#include <${inclusion_prefix}/component>` format.. 
set( inclusion_prefix "mcl/core" )

add_library( core ) 
target_sources( core
	PRIVATE
		someComponent.cpp
		# Other translation units ...
	PUBLIC
		FILE_SET coreHeaders
		TYPE HEADERS
		BASE_DIRS
			${CMAKE_CURRENT_SOURCE_DIR}/../..
			${CMAKE_CURRENT_BINARY_DIR}
		FILES
			someComponent.hpp
			${CMAKE_CURRENT_BINARY_DIR}/${inclusion_prefix}/someGeneratedHeader.hpp
)
```


Note that `${CMAKE_CURRENT_BINARY_DIR}` is added directly to `BASE_DIRS` while
`${CMAKE_CURRENT_SOURCE_DIR}` is not. Instead, the latter's grandparent is added
to `BASE_DIRS`. We can add the former directly because we have complete control
over the directory structure of the build area. Creating additional levels of
nesting by appending the inclusion prefix is no problem; only the developer
building the library is affected.

For the `CMAKE_CURRENT_SOURCE_DIR` case, specifying `CMAKE_CURRENT_SOURCE_DIR`
would either change the include directive `#include <someComponent.hpp>`, **or**, if
the `<mcl/core/someComponent.hpp>` is still desired, require additional nesting
in the project tree. This nesting would result in a
messier project structure:

```
mcl/
|
├── core/
|	├── mcl/core/
|	|	|		
|	|	├── someComponent.cpp
|	|	├── someComponent.h
|	|	└── ...
|	└── CMakeLists.txt
├── ...
└── CMakeLists.txt
```

Instead, we move *up* two levels to specify that the directory containing the
overall `mcl` project is added to the header search path. This is valid because:

1. `mcl` will never be the root directory of a file system. It always has a
   parent.
2. The build directory is neither a sibling to the project source tree nor a
   child of the project source tree.  
	- This is required because "No two base directories for a file set may be
	sub-directories of each other." Since `mcl`'s parent directory is added to
	the set, the build area cannot live anywhere on that branch. 

## Relevant Reading

- [Using Dependencies
  Guide](https://cmake.org/cmake/help/latest/guide/using-dependencies/index.html#guide:Using%20Dependencies%20Guide)
- [`target_sources`](https://cmake.org/cmake/help/latest/command/target_sources.html)
- [`target_link_libraries`](https://cmake.org/cmake/help/latest/command/target_link_libraries.html)