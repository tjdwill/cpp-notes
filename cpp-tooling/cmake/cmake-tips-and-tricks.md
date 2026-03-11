# CMake Tips and Tricks

These are (unordered) recommended practices and small tips I've learned from reading or
experimentation.

- Periodically change the generator used for building to ensure generator-independence. This
  encourages good cross-compilation practices.

- `set(CMAKE_EXPORT_COMPILE_COMMANDS ON)` generates a `compile_commands.json` file that can then be
  used for tools like `clangd`.
    - It's helpful to create a (relative) symbolic link to the file to keep it updated: 

    ```bash
    # in project root
    ln -rs build/compile_commands.json .
    ```

- If the project build is scripted, use `cmake --build` instead of invoking the build tool directly
  (ex. `ninja`). 
    - Rationale: Facilitates easy switching between generator types.

- In lieu of `CMAKE_SOURCE_DIR` and the like, use the `PROJECT_SOURCE_DIR` line of variables for
  pathing for projects that may be included as a child to another CMake project. This will allow for
  the proper paths to be derived.

- WINDOWS: by default, Windows applications will open a cmd window. If your application is a GUI
  executable, you can disable this behavior by specifying the
  [`WIN32_EXECUTABLE`](https://cmake.org/cmake/help/latest/prop_tgt/WIN32_EXECUTABLE.html#win32-executable)
  property on your target. This property does not affect behavior on other platforms.

## Dynamic Compilation Database Linking

Sometimes, you may want to keep the top-level `compile_commands.json` up-to-date with whatever
config preset you are building with. To do so, we use `execute_process()`:

```cmake
if (CMAKE_HOST_SYSTEM_NAME STREQUAL "Linux")
  execute_process(
      COMMAND ${CMAKE_COMMAND} -E create_symlink
          ${CMAKE_BINARY_DIR}/compile_commands.json
          ${CMAKE_CURRENT_SOURCE_DIR}/compile_commands.json
  )
endif()
```

Important to note that this command expects a list of strings instead of one
long string.

Windows doesn't have symbolic link support in the sense of Unix symbolic links. To get around this, I create a custom target that copies the `compile_commands.json` from the relevant build directory to the top-level directory:

```cmake
elseif( CMAKE_HOST_WIN32 )

  add_custom_target(copyCompilationDatabase ALL

    COMMAND powershell "Copy-Item -Path ${CMAKE_BINARY_DIR}/compile_commands.json -Destination ${CMAKE_SOURCE_DIR}/compile_commands.json"
    VERBATIM
  )
```

I then set `clangd`'s `compile-commands-dir` argument to point to the top-level project directory.

READ MORE:

 - https://cmake.org/cmake/help/book/mastering-cmake/chapter/Custom%20Commands.html
 - https://stackoverflow.com/a/59264495
 - https://cmake.org/cmake/help/latest/command/execute_process.html

## Copying Runtime Dependencies to a Specified Directory
*Motivation*: For debugging purposes, it is useful to have a given target's runtime dependencies in desirable locations. This is especially relevant in cases where a target A dynamically links to a shared library B that's built in a different subfolder of the top-level project. Additionally, our tests may depend of files that exist in a specific relative location.

We leverage [`add_custom_command()`](https://cmake.org/cmake/help/latest/command/add_custom_command.html) for this purpose. 

Say we have some target `MyProject::Foo` (assuming some target alias in namespace `MyProject`) that depends on `MyProject::Bar` defined in a different subdirectory.

Assuming `MyProject::Bar` is a shared library `MyProject::Foo` links to, we can write the following:

```cmake
# In the CML that defines MyProject::Foo
## find_package() or add_subdirectory() such that MyProject::Bar is defined in this scope.
add_executable( Foo )
add_executable( MyProject::Foo ALIAS Foo )
target_link_libraries( Foo PRIVATE MyProject::Bar )
# Target Configuration
# ...
# Copy Foo's dependency DLLs into a common location.
add_custom_command(TARGET Foo
	POST_BUILD
	
	COMMAND ${CMAKE_COMMAND} -E copy -t ${CMAKE_CURRENT_BINARY_DIR} $<TARGET_RUNTIME_DLLS:MyProject::Foo>
	
	VERBATIM
	COMMAND_EXPAND_LISTS
)
```

The above command defines a custom command associated with the `Foo` target that runs *after* the Foo target rules have run (`POST_BUILD`). We use the CMake executable referenced via the `${CMAKE_COMMAND}` variable and the `-E` option to run a [command-line tool](https://cmake.org/cmake/help/latest/manual/cmake.1.html#run-a-command-line-tool). Specifically, we copy the specified files to the current binary dir.

With a combination of `COMMAND`, the included command line tools via `${CMAKE_COMMAND} -E`, and [target-related generator expressions](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#target-artifacts), we have freedom needed to customize our build to facilitate testing or runtime debugging.