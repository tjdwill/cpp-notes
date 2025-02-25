# cpp-notes

> A place to document my C++ journey

**Note**: While the intention here is to document my C++ trajectory, it may also bleed into general Computer Science learning. Things like DSA, concurrency, memory-managment, abstraction, etc. are universal concepts in programming.

---

Learning C++ is difficult. There are conflicting "styles" (or rather "epochs") in the language such that two sets of competent programmers can work in a completely different, possibly conflicting ideas of what is acceptable. Bottom line, C++ is hard, and it is more reasonable to aim for competence rather than mastery. I think as long as I can achieve my goals using solid software design principles, data organization, and algorithms, I will be content. That being said, here is a list of topics I want to learn in order to learn C++:

- Syntax (keywords, expressions, functions, classes, etc.)
- Semantics
    - const correctness
    - value semantics vs. move semantics
    - Inheritance vs. composition
- Resource Management and Safety
    - RAII
    - Lifetimes
    - Memory model
- Error Handling
    - Exceptions vs. Algebraic Data Types (ADTs as error codes)
- Data Structures and Algorithms
- Standard Library
    - containers (map, array, vector, set, etc.)
    - algorithms
    - important types (string, chrono types, ADTs)
    - utilities (ex. iostreams, files, asynchronous communication, etc.)
- Third-Party Libraries
    - `Eigen`
    - `OpenCV`
    - `ICU` (for UTF-8)
    - `Qt`
- Tooling 
    - Build System
        - Basics (manual compilation; installing and linking third-party libraries locally)
        - `make`, `CMake`, and `Ninja` (+ possibly `Meson`?)
        - Package Management (`vcpkg` or `Conan`)
        - Compilers (`gcc`, `clang`, and (opt.) `Visual Studio` for windows)
    - Linting and Language Server Protocol
        - `clang-tidy` & `clangd`
    - Debugging and Profiling
        - `gdb`
        - `valgrind`
- Software Design
    - John Lakos' Work
    - Klaus Iglberger
    - Scott Meyers
    - How to structure projects logically and physically?
    - Long term maintainence?
    - How to release a project?
    - LICENSING?

So far, I've learned that it's infeasible to work at the above (or any other set of
knowledge) linearly, mastering each topic in sequence. Instead, an iterative approach is
necessary. For one, a more well-rounded progression is achieved. Secondly, it is important
to *feel* like I'm progressing in order to remain motivated. The iterative approach will
enable this, allowing me to tackle problems of increasing complexity.

In order to do this, however, practical experience is critical as a vehicle for exploring
and solidifying academic learning. Therefore, projects are a must. Small projects are
useful for exploring a concept or refining the understanding of another. Medium to large
projects are stretch projects that provide practice in engineering and design in addition
to advancing in some field of application.

## Subdomains

Speaking of which, it's important to have use cases for the language, domains in which I
wish to utilize C++ for some end. My primary motivation is robotics due to C++
being the *de facto* "performant" language in the field. However, there are transferrable
skills in the following domains I wish to explore:

- Asynchronous Programming
- Event Handling
- Data Processing
- Parsing
- Networking & Communications
- User Interface Design (CLIs, GUIs, and APIs)
