Date: 16 October 2025

One frustrating aspect of CMake is the sheer number of ways to achieve the same desired result. Exacerbating this issue is that every developer seems to have a different idea of what true "modern" CMake looks like. As we all twirl our mustaches and adjust our monocles in pursuit of this Platonic ideal, here are some notes from an exchange I had with an actual CMake developer (Redditor /u/not-a-novel-account, A.K.A nickelpro) about includes (via FILE_SET).

First, let's talk about header ownership because this concept confused me initially. A given target , a named collection of properties typically declared by `add_[library|executable]()`, owns a given header file if that target declares and defines entities in said header as part of the target's interface. This really only makes sense if the target is a library, so let's assume that's the case. Other targets can *use* that header (think of Rust's "borrow" concept), but the header *belongs* to the owning target. To quote the helpful CMake dev directly:

> If library Alpha has a function `foo()`, then the header with the declaration for `foo()` belongs to Alpha. If library Bravo has a function `bar()`, the header declaring `bar()` belongs to Bravo. The header for `bar()`, say `bar.hpp`, should not appear in the `target_sources()` command for library Alpha.

If the header is part of the library's interface, it should be associated with that library target via `target_sources()`.  We define the list of headers owned by a target via the `FILE_SET` portion of `target_sources()`. This directive allows developers to specify both *which* headers are owned by a target as well as *where* to find them (see `BASE_DIRS` and `FILES`). Files in a `PRIVATE` set are needed to build the given target; `INTERFACE` files are required to consume the parget. `PUBLIC` files are for both. By defining the FILE_SET for a target, dependents can inherit the include paths associated with the target via `target_link_libraries()`.

So, each target is responsible for its own includes. If a given target X wishes to interface with another target Y's includes, X must link to the Y via `target_link_libraries()`. This will allow X to inherit Y's include paths without having to specify them in `target_sources()`. Based on this paradigm, it would appear that the preferred way of consuming third party entities is to:

0. (Build and) Install the given package via CMake (look into `target_include_directories()` and other alternatives for non-CMake projects). 
1. Import the package in the relevant CMakeLists.txt (CML) via `find_package()`.
2. Link our desired target to the relevant imported package target.
	- One may need to read the third party package's CMake in order to ascertain the correct naming conventions to use for the targets.

## Relevant Reading

- [Using Dependencies Guide](https://cmake.org/cmake/help/latest/guide/using-dependencies/index.html#guide:Using%20Dependencies%20Guide)
- [`target_sources`](https://cmake.org/cmake/help/latest/command/target_sources.html)
- [`target_link_libraries`](https://cmake.org/cmake/help/latest/command/target_link_libraries.html)