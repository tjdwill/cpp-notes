# CMake Tips and Tricks

These are (unordered) recommended practices and small tips I've learned from reading or experimentation.

- Periodically change the generator used for building to ensure generator-independence. This encourages good cross-compilation practices.

- `set(CMAKE_EXPORT_COMPILE_COMMANDS ON)` generates a `compile_commands.json` file that can then be used for tools like clangd.
    - It's helpful to create a (relative) symbolic link to the file to keep it updated: 

    ```bash
    # in project root
    ln -rs build/compile_commands.json .`
    ```

- If the project build is scripted, use `cmake --build` instead of invoking the build tool directly (ex. `ninja`). 
    - Rationale: Facilitates easy switching between generator types.
