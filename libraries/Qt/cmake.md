# Build an Application with CMake

Many projects use CMake as their primary build system, so, for the sake of uniformity, it
may be desirable to build Qt using CMake instead of qmake. Here is a bare-bones template
that assumes Qt5.

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

## UIC Note

Something to note is that when using auto-generated classes from QtDesigner (UIC),
transient dependence on the generated header file doesn't work. It results in the following
error:

```console
In file included from /home/tj/programming/cpp/qt-practice/src/ch02/gotocelldialog.t.cpp:2:
/home/tj/programming/cpp/qt-practice/src/ch02/gotocelldialog.h:5:10: fatal error: ui_gotocelldialog.h: No such file or directory
    5 | #include "ui_gotocelldialog.h"
      |          ^~~~~~~~~~~~~~~~~~~~~
compilation terminated.
gmake[2]: *** [src/ch02/CMakeFiles/test_gotocelldialog.dir/build.make:93: src/ch02/CMakeFiles/test_gotocelldialog.dir/gotocelldialog.t.cpp.o] Error 1
gmake[1]: *** [CMakeFiles/Makefile2:448: src/ch02/CMakeFiles/test_gotocelldialog.dir/all] Error 2
gmake: *** [Makefile:91: all] Error 2
```

The solution was to either write the include directive (*i.e.* `#include "ui_<whatever the
name is>.h`) directly in the file in question or to list the file that already has the
direct inclusion as a target dependency in the `add_executable` command.


## RCC Note

When using resources for your application, you need to add the `.qrc` file as a target
dependency in the formula generating *the application*. So basically, when you finally
create the executable, add the relevant `.qrc` files to the `add_executable()` command.

## AUTOMOC Note

Reference: https://stackoverflow.com/a/49998741

If defining a class in a `.cpp` file that inherits from a Qt class, we need to include the
generated `moc` file manually in order to compile successfully. 

For some `foo.cpp`, you would include `#include "foo.moc"` **after** all of the
`Qt`-specific structures are defined. The above source recommends doing so at the end of
the file.

## Qt Keywords

Qt provides the ability to use `emit` and others as keywords. While I initially preferred this to the macro versions (ex. `Q_EMIT`), but, after coming across a symbol collision in which a third party library *also* used some function with named `emit` something-or-another (the third party developers were kind enough to fix this after I submitted a bug report), I now prefer the macros.  

We can enforce the use of the macros in a project by setting the `QT_NO_KEYWORDS` variable.
This can be done, for example, by `add_compile_definitions(QT_NO_KEYWORDS)`