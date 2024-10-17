# My C++ Journey

Without mincing words, learning C++ is difficult. There are conflicting "styles" (or rather "epochs") in the language such that one set of competent programmers work in a completely different, possibly 


- Qt: Download
    - aqtinstall
    - mesa (OpenGL impl)
        - `sudo apt install mesa-common-dev libglu1-mesa-dev`
        - https://stackoverflow.com/questions/58787687/qt-5-12-failed-to-find-gl-gl-h-in-usr-include-libdrm
    - Dracula theme
- CMake: 
    - Set `CMAKE_PREFIX_PATH`
    - (IF not using QtCreator)
        - Set `CMAKE_EXPORT_COMPILE_COMMANDS ON`
        - Symlink `compile_commands.json` to top-level
        - In `.clangd`, set the `CompilationDatabase` entry to the directory of the real `compile_commands.json` file (i.e. wherever your CMake build folder is).
        - In `CMakeLists.txt`, use [the tutorial](https://doc.qt.io/qt-6/cmake-manual.html) to ensure you have find Qt correctly.

