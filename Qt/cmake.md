# Build an Application with CMake

Many projects use CMake as their primary build system, so, for the sake of uniformity, it may be desirable to build Qt using CMake instead of qmake. Here is a bare-bones template that assumes Qt5.

```cmake
cmake_minimum_required(VERSION 3.20)
project(myproject VERSION 0.0.1 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Generate compile_commands.json for clangd
# outputs file in the project's specified build directory
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Qt config
## Handles file generation
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

# Search for Qt installation. If installed locally, be sure CMAKE_PREFIX_PATH is defined
# and that the Qt installation path is on it.

find_package(Qt5 REQUIRED COMPONENTS Widgets)  # add any other Qt components used.

add_executable(myQtApp
    main.cpp
)
target_link_libraries(myQtApp
    PUBLIC 
    Qt5::Widgets
)
```
