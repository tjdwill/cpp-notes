# Common C++ Tools

In addition to learning the language, becoming a competent C++ developer requires learning
the ecosystem. That is, the tools and processes used to program and build the C++ in the
first place. Being an older language with no standardized build system, creating a project
in C++ has a lot more moving parts and things to learn:

- Build System
    - CMake (*de facto* build tool)
    - Ninja (generator)
    - Compiler/Linker
    - eventually... Package Manager (vcpkg; Conan)
- Debugger
    - gdb, lldb; Windows alternatives (Visual Studio; windbg)
    - valgrind
    - Sanitizers (Address Sanitizer)
- Formatters and Linters (clangd, clang-tidy)
- Editors
    - VSCode
    - Neovim
    - Visual Studio
- Version Control
    - Git (w/ Pre-commit hooks)

In a given project, I have used something like:

- CMake + Ninja
- clang++
- vcpkg
- clangd and clang-tidy
- VSCode or NeoVim
- Git
- pre-commit

The iterative approach is key for me. Time spent on one tool fills gaps in knowledge that
then make using and integrating the other tools easier.
