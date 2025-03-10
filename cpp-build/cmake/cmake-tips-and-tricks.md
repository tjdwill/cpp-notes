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

Place 

READ MORE:
 - https://cmake.org/cmake/help/book/mastering-cmake/chapter/Custom%20Commands.html
 - https://stackoverflow.com/a/59264495
 - https://cmake.org/cmake/help/latest/command/execute_process.html
