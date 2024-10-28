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
