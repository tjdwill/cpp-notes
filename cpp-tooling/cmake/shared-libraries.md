When creating a library target via `add_library()`, users have the option to specify the linking type. For static libraries, specify `STATIC`. Shared libraries can be built via `SHARED`.  By default, libraries are built statically unless `BUILD_SHARED_LIBS` is set to `ON`. When building with shared libraries, however, there are extra steps that must be taken for the program to build and link properly.

##  Symbol Visibility

Linkers deal with symbol resolution, associating the names of functions, variables, classes, etc. with their definitions. Consumers of static libraries each get their own copy of the code; the symbols and code definitions are embedded into the target executable. Shared libraries are more like references that must be resolved. However, in order for symbols to be discoverable, they must be visible to consumers (similar to how a method should be `public` for consumers to use them indiscriminately). 

Symbol visibility defaults vary by platform. On Linux, GCC and Clang make symbols visible by default, meaning consumers can easily and readily use entities defined in a shared library. MSVC, however, makes symbols hidden by default; no entities in the shared library are accessible to external entities. In fact, as we'll see later, the build process will likely fail at link time when attempting to like with shared library with zero visible symbols. 

To make a symbol visible, the developer must *export* it. Different compilers have different-but-similarly-functioning mechanisms for doing so. The familiar one in a Windows environment is the `__declspec(dllexport)` keyword that exports the symbol and the `__declspec(dllimport)` keyword that imports it. Bottom line, relevant symbols must be made visible in order to use them via a shared library context, so let's see how we can do so in CMake.


### Export Headers

To make the symbol visibility handling easier, developers utilize conditionally-defined macros that automatically expand to, for example, one of the two `__declspec` keywords based on whether the library is being built or consumed. These macros can be generated automatically with CMake through the use of the `GenerateExportHeader` module. With is flagship command, [`generate_export_header`](https://cmake.org/cmake/help/latest/module/GenerateExportHeader.html#command:generate_export_header) and the use of the FILE_SET feature, users can easily mark their symbols as visible or hidden as desired. Here is an example from my [DBS](https://githuub.com/tjwill/dbs) project:

```cmake
include(GenerateExportHeader)
add_library(dbsc)
generate_export_header(dbsc
  EXPORT_FILE_NAME dbsc_sharedapi.h
  EXPORT_MACRO_NAME DBSC_API
  NO_EXPORT_MACRO_NAME DBSC_INTERNAL
)
target_sources(dbsc
  PRIVATE
    dbsc_uuidstring.cpp
    dbsc_transaction.cpp
    dbsc_account.cpp
    dbsc_accountbook.cpp
    dbsc_dbscserializer.cpp
    dbsc_tomlserializer.cpp
  PUBLIC
    FILE_SET HEADERS
    BASE_DIRS
      ${CMAKE_CURRENT_SOURCE_DIR}
      ${CMAKE_CURRENT_BINARY_DIR}
    FILES
      dbsc_registerexception.h
      dbsc_uuidstring.h
      dbsc_transaction.h
      dbsc_account.h
      dbsc_accountbook.h
      dbsc_dbscserializer.h
      dbsc_tomlserializer.h
      ${CMAKE_CURRENT_BINARY_DIR}/dbsc_sharedapi.h
)
target_link_libraries(dbsc
  PUBLIC
    bdl
    bsl
    stduuid
    tomlplusplus::tomlplusplus
)
install(TARGETS dbsc)
```
We see in the `generate_export_header()` call a specification of the filename of the  generated header file containing the visibility macros as well as the names of the desired macros themselves. Since CMake generates this file, it lives in the build directory. Therefore, when the  header is added to a FILE_SET, it must be specified as living in the relevant binary directory. 

Using these macros are simple enough. Simply include the header and mark the desired classes, functions, and variables:


```cpp
// someheader.h
// Specifying visiblity (class, function, variable)
#include <some_sharedapimacros.h>

struct VISIBILITY_MACRO SomeType{};
[[nodiscard]] VISIBILITY_MACRO auto someFunction() -> bool;
VISIBILITY_MACRO const auto kThisIsAGlobalVariable = 3;
```

Entities that are fully defined in a header (templates, POD structs, etc.) need not be exported. Their definitions are accessible directly via the header.

## Windows Linking Error

Windows is a bit unusual in that it produces two files when building a shared library. There is the expected `.dll` file that is used for runtime loading, but there is also the `.lib` file that is used 
for linking during program building. 

It's possible to receive a linking error when building/consuming a shared library that states `someComponent.lib` could not be found. To produce a `.lib` file, Windows expects at least one symbol to be exported from the shared library. If shown this error, your library likely doesn't have any exported symbols. 