Date: 6 February 2026

- [Documentation Link](https://cmake.org/cmake/help/latest/command/install.html)
- Scott *Professional CMake*: Ch. 27 "Installing"
## Basic Installation

To "install" a program means to place the relevant files associated with the project in a known location that is potentially user-specified. In CMake, installation rules are created through use of the `install()` command.

`install()` has many signatures that affect what rules are created. The most common form is the `install(TARGETS)` signature. At its most basic, CMake will generate rules to install the output artifacts (shared libraries, executables, static libraries, etc.) in expected locations. For example, given some library Alpha:

```cmake
add_library( Alpha ) # Defines target 'Alpha'
# ... target_sources, target_link_libraries, target_compile_features, etc.
install( TARGETS Alpha )
```


the install command will place the artifacts of the `Alpha` target into default locations when `cmake --install <directory>` is invoked.

- As of 3.14, `install()` commands in CMLs added via `add_subdirectory()` are interleaved with those in the parent directory such that the install rules execute in declaration order.

## Specifying Install Locations

CMake utilizes the locations specified by the [`GNUInstallDirs`](https://cmake.org/cmake/help/latest/module/GNUInstallDirs.html) module.  The location a given file is installed to depends on its [artifact kind](https://cmake.org/cmake/help/latest/command/install.html#signatures), a superset of the [output artifact concept](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#output-artifacts). There are a number of artifact kinds, but the most commonly used are `RUNTIME`, `ARCHIVE`, `LIBRARY`, and `FILE_SET`. 

Each artifact kind can have a number of options customized such as `DESTINATION`. By specifying, for example,

```cmake
include( GNUInstallDirs )
install( TARGETS foo
	RUNTIME
		DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
```

one could have runtime artifacts (and other unspecified artifact kinds)  placed in the `include` directory.

`GNUInstallDirs` directories are relative to `CMAKE_INSTALL_PREFIX`, which can be specified by `cmake --install <buildDir> --prefix=<Desired Installation Directory>`.

## NSIS

See the [NSIS Download Page](https://nsis.sourceforge.io)

## Exports

### Target Aliases, Export Names, and Namespaces

### Multi-Export Builds

## Creating a Config File

## 