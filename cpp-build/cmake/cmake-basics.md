# CMake Basics

> Attempting to use any tool before understanding at least the basics of what it does and how it is meant to be used is most likely going to result in frustration. On the other hand, spending all one’s
time learning the theory about something without getting hands-on makes for a rather boring experience and often leads to an overly idealistic understanding.
    \- Craig Scott. *Professional CMake: A Practical Guide*

**Basic Flow**

    Project File Generation -> Build -> Test -> Package
    [Configure -> Generate]                

The central component of CMake is the `CMakeLists.txt` file. It is a platform-independent file that is processed by CMake to produce platform-specific build instructions. It is recommended ot use CMake as an *out-of-source* build system, meaning users should distinguish between a *source* directory and a *binary* (build) directory. There are [many benefits to doing so](https://johnfarrier.com/in-source-vs-out-of-source-builds/), but the basic one is that you can safely delete the build folder when desired. It also enables the ability to create multiple, distinct builds (ex. debug vs. release). 

`CMakeLists.txt` is placed in the source directory, but I've also seen it placed in the project's root directory. Regardless, the book recommends that the build directory is kept separate from the source. The file should be placed under version control along with sources.


**Basic Project Structure**

```
Project
├── source
│     ├── CMakeLists.txt
│     └── ... sources
│
└── build
      ├── CMakeCache.txt
      └── ... build output files
```

Another version I've seen moves the CMakeLists.txt file to be under the root directory.

Whereas typical build systems like `make` are used to build recipes that generate object files and the like, `CMake` is more of a system that generates files used by build systems. It can generate files for make, Ninja, XCode, Visual Studio, etc. The developer selects the *generator* as part of configuration.

You can specify the project file generator using the `-G` option of the CLI tool.


## Basic Example

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.20)
project(MyProject)
add_executable(App main.cpp)
```

The commands above are like function calls. Commands in CMake do not return values. Arguments are whitespace-delimited and can be separated by newlines. As an aside, command names are case-insensitive.

In order to configure the proper policies, the first line of *any* top-level `CMakeLists.txt` should be the [`cmake_minimum_required()`](https://cmake.org/cmake/help/latest/command/cmake_minimum_required.html#cmake-minimum-required) command. Next is the [`project()`](https://cmake.org/cmake/help/latest/command/project.html#project) command.

## Comments

- Single-line comment: `#`
- Multi-line comment: `#[[ comment here ]]`

## Linking Types

For a given target (via [`add_executable()`](https://cmake.org/cmake/help/latest/command/add_executable.html) or [`add_library()`](https://cmake.org/cmake/help/latest/command/add_library.html)), we can specify libraries to link using [`target_link_libraries`](https://cmake.org/cmake/help/latest/command/target_link_libraries.html). This command has multiple keywords that specify the level of dependency:

- PRIVATE: the library is only used internally as an implementation detail. The dependency isn't exposed to the user.
- PUBLIC: the library is required to use the target (used internally and in interface)
- INTERFACE: Components of the library are used in the interface of the target.

## Bring in Subdirectories

We can organize our build process into a tree of `CMakeLists.txt` files. Meaning, we don't have to put everything in our top-level CMakeLists; we can modularize our build, importing the submodules as desired (note here that I am not referring to CMake's concept of modules; I simply mean the breaking the build into subcomponents).

To do so, we have (at least) two options: `add_subdirectory()` and `include()`. 

### `add_subdirectory()`

This function executes the contents of a CMakeLists.txt file located in some specified directory. Typically, this directory is a sub-directory of the project's source tree. The path to said directory can be either absolute or relative to the current *source* directory (see [the special variables section](#special-variables)). 

`add_subdirectory()` creates a new scope when processing the relevant CMakeLists file. This scope is a child scope to the calling scope. The rules regarding variables are similar to those for `block()` invocations. 

1. The child gets a copy of all variables from the calling scope.
2. Variables created in the child scope are local to that scope (not visible to parent).
3. Changes to a variable in child scope are local to said scope (unless `PARENT_SCOPE` is specified).

Note that it is unnecessary to call `project()` in a nested CMakeLists.txt file. 

### `include()`

According to [this link in the CMake email list](https://cmake.org/pipermail/cmake/2007-November/017897.html), `include()` is used similarly to `#include` in a C/C++ context. If we want, for example, to make common utility functions or target recipes availabe to the entire project, we'd include the relevant `.cmake` file at the root.

That being said, it should be clear that the command expects a file path in contrast to `add_subdirectory()` expecting a directory path. Another form of `include()` accepts a module name, but the book postpones that topic for later.

Included files don't *have* to be `.cmake` files, but they typically are. 
Unlike `add_subdirectory()`, `include()` does *not* begin a new variable scope (though both begin new policy scopes), nor does it change the current source and binary directories. As a result, CMake provides three additional read-only variables (see the [special variables section](#special-variables). These variables provide the directory path of the included file, the file's path, and the line number of the inclusion operation in the current CMakeLists file.

## Special Variables

The following four path variables always resolve to absolute paths:

- `CMAKE_SOURCE_DIR`: the top-most directory of the *source* tree (where the top-level CMakeLists.txt is located). Never changes in value.
- `CMAKE_BINARY_DIR`: the top-most directory of the *build* tree. Never changes in value.
- `CMAKE_CURRENT_SOURCE_DIR`: the directory of the current CMakeLists.txt being processed. This changes with calls to `add_subdirectory()`.
- `CMAKE_CURRENT_BINARY_DIR`: the build directory corresponding to the current CMakeLists.txt being processed. Changes with calls to `add_subdirectory()`.

- `CMAKE_CURRENT_LIST_DIR`: Get directory of the listfile being processed (abs path)
- `CMAKE_CURRENT_LIST_FILE`: Get absolute path of the listfile.
- `CMAKE_CURRENT_LIST_LINE`: Current line number of the file being processed.

- `PROJECT_SOURCE_DIR`: The source directory of the most recent call to `project()` in the current scope or any parent scope.
- `PROJECT_BINARY_DIR`: Build directory analogue to `PROJECT_SOURCE_DIR`
- `projectName_SOURCE_DIR`: The source directory of the most recent call to `project(projectName)` in the current scope or any parent scope.
- `projectName_BINARY_DIR`: Build directory analogue to `projectName_SOURCE_DIR`.


The following variables are only compatible with CMake 3.21 or later.
- `PROJECT_IS_TOP_LEVEL`: determines if the most recent call to `project()` was in the top level CMakeLists file.
- `<project_name>_IS_TOP_LEVEL`: Cache variable. Similar function to `PROJECT_IS_TOP_LEVEL`


### Useful Cache Variables 

For variables I know I won't remember.

- `CMAKE_BUILD_TYPE`: Sets the build type for the project (ex. Debug or Release)
  
    ```bash
    - cmake -G Ninja -DCMAKE_BUILD_TYPE:String=Debug <src_path>
    ```

## Properties

- `INCLUDE_DIRECTORIES`: The list of directories used as header search paths. Paths must be absolute.
- `COMPILE_DEFINITIONS`: I think this is a list of pre-processor macros?
- `COMPILE_OPTIONS`: Any compiler flag that is neither a header search path nor a symbol definition.
