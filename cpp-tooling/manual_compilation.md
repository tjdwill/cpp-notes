# Manual Compilation In C++

Date: 24 November 2025

The build process for C++ was both daunting and enigmatic when I began learning the language. Now, after further
experience with both the language and Linux, the process is much more palatable,
at least for the basics. In this article, I will walk through compiling a complete program,
including linking with an external library.

## Sources

We're going to compile a simple program that uses a "third-party" library to add and
subtract integers. Here is the program:

```cpp
// main.cpp
#include <add.hpp>
#include <subtract.hpp>

#include <iostream>
#include <stdexcept>

int main() {
  if (cppbuild::add(1, 3) != 4 || cppbuild::subtract(3, 3) != 0) {
    throw std::runtime_error("Incorrect add or subtract implementation.");
  }

  std::cout << "Success!" << "\n";

  return 0;
}
```

We use an external library, `cppbuild` that provides functions `cppbuild::{add, subtract}`.
Here are the implementations:

```cpp
// add.hpp
#ifndef INCLUDED_CPPBUILD_ADD
#define INCLUDED_CPPBUILD_ADD
//@PURPOSE: Provide an addition function to serve as a linkable library

namespace cppbuild {
/// @return the addition of the two inputs
auto add(int x, int y) -> int;
} // namespace cppbuild

#endif // header include guard

//-----------------------------------------------------------------------------
// add.cpp
#include "add.hpp"

auto cppbuild::add(int const x, int const y) -> int { return x + y; }

//-----------------------------------------------------------------------------
// subtract.hpp
#ifndef INCLUDED_CPPBUILD_SUBTRACT
#define INCLUDED_CPPBUILD_SUBTRACT
// @PURPOSE: Implement a dompon3n5 5hq5 dq b3 dombni==in3e
namespace cppbuild {
auto subtract(int x, int y) -> int;
}
#endif // header include guard

//-----------------------------------------------------------------------------
// subtract.cpp
#include "subtract.hpp"
auto cppbuild::subtract(int const x, int const y) -> int { return x - y; }
```

The project will be referenced as `$projectDir`
in command line invocations. It has the following structure:

```
cppbuild
├── build
└── src
    ├── add.cpp
    ├── add.hpp
    ├── main.cpp
    ├── subtract.cpp
    └── subtract.hpp
```

Note that even though the `add` and `subtract` components are in the same folder as `main`,
we'll compile as if they are part of a separate library.

## Build Process

The C++ Standard specifies nine steps in the build process.
However, in practice, some of those steps are folded together, happening in the same phase.
For our purposes, here are the steps to build a C++ program:

1. *Preprocessing*: macro expansion, include processing, comment removal, etc.
2. *Compilation*: Processes pre-processed code, producing assembly for the target
   architecture.
3. *Assembly*: Converting from assembly to binary machine code. 
4. *Linking*: Resolving symbols to associate all symbol declarations and uses with its definition, producing an executable.

### Viewing preprocessor output

One can print preprocessor output to the screen by using the `-E` option flag (side note:
I'm using clang):

```console
cppbuild> clang++ -E src/add.hpp
```

More options can be found in [the preprocessor options
page](https://clang.llvm.org/docs/ClangCommandLineReference.html#preprocessor-options).

### Viewing compilation assembly

You can stop the build process after the compilation step by specifying the `-S` flag. This
produces a `.s` file that shows the assembly produced from compilation.

```console
cppbuild> mkdir build && cd build
cppbuild/build> clang++ -S $projectDir/src/add.cpp
cppbuild/build> cat add.s
	.text
	.file	"add.cpp"
	.globl	_ZN8cppbuild3addEii             # -- Begin function _ZN8cppbuild3addEii
	.p2align	4, 0x90
	.type	_ZN8cppbuild3addEii,@function
_ZN8cppbuild3addEii:                    # @_ZN8cppbuild3addEii
	.cfi_startproc
# %bb.0:
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset %rbp, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register %rbp
	movl	%edi, -4(%rbp)
	movl	%esi, -8(%rbp)
	movl	-4(%rbp), %eax
	addl	-8(%rbp), %eax
	popq	%rbp
	.cfi_def_cfa %rsp, 8
	retq
.Lfunc_end0:
	.size	_ZN8cppbuild3addEii, .Lfunc_end0-_ZN8cppbuild3addEii
	.cfi_endproc
                                        # -- End function
	.ident	"Debian clang version 14.0.6"
	.section	".note.GNU-stack","",@progbits
	.addrsig

```

### Viewing the object file

We can stop at producing the binary object file `.o` using the `-c` flag:

```console
cppbuild> cd build
cppbuild/build> clang++ -o add.o -c $projectDir/src/add.cpp
cppbuild/build> objdump -d add.o

add.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <_ZN8cppbuild3addEii>:
   0:	55                   	push   %rbp
   1:	48 89 e5             	mov    %rsp,%rbp
   4:	89 7d fc             	mov    %edi,-0x4(%rbp)
   7:	89 75 f8             	mov    %esi,-0x8(%rbp)
   a:	8b 45 fc             	mov    -0x4(%rbp),%eax
   d:	03 45 f8             	add    -0x8(%rbp),%eax
  10:	5d                   	pop    %rbp
  11:	c3                   	ret

```

## Building the `cppbuild` Static Library (`libcppbuild.a`)

We'll first be building a static library. This archives all of object files in the
`cppbuild` library, allowing the binary code to be copied into the resulting compiled
executable.

First, we need to compile the individual library object files:

```console
cppbuild/build> clang++ -o add.o -c $projectDir/src/add.cpp
cppbuild/build> clang++ -o subtract.o -c $projectDir/src/subtract.cpp
```

Next, we archive the object files using the `ar` command (learn more via `man ar`):

```console
cppbuild/build> ar rcs libcppbuild.a add.o subtract.o
```

And that's it! Now, since we followed the static library naming convention, we can link to the `cppbuild` library.

## Building the `cppbuild` Shared Library (`libcppbuild.so`)

To build the `cppbuild` as a shared library, we change the compilation of the individual
components to be position independent using the `-fPIC` flag:

```console
cppbuild/build> mkdir so # Shared library directory
cppbuild/build> clang++ -o so/add.o -c $projectDir/src/add.cpp -fPIC
cppbuild/build> clang++ -o so/subtract.o -c $projectDir/src/subtract.cpp -fPIC
```

Next, we coalesce the binaries into a shared object file:

```console
cppbuild/build> clang++ -o so/libcppbuild.so -shared add.o subtract.o
cppbuild/build> ls -lhgFG so
total 24K
-rw-r--r-- 1 976 Nov 24 21:12 add.o
-rwxr-xr-x 1 16K Nov 24 21:12 libcppbuild.so*
-rw-r--r-- 1 984 Nov 24 21:12 subtract.o
```

Now, we need to register the shared library in order for it to be found by the loader at
runtime. Assuming administrator access, run the following:

```console
cppbuild/build> sudo ldconfig $projectDir/build/so
```

This adds the shared libraries under the input directory to the list of discovered libraries. Restart your shell.

If you don't have super user access (`sudo`), edit the `LD_LIBRARY_PATH` environment
variable:

```console
cppbuild/build> export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$projectDir/build/so
```

## Compiling `main`

To successfully compile `main`, we need to ensure the headers and linked binaries are
discoverable by adding the relevant directories to the compiler's search paths. For headers, this is done via the `-I` compiler flag. For libraries, this is done via the `-L` flag. Finally, the library to link against is specified via the `-l` flag. If a library has filename `libmyLibrary.{a/so}`, the link directive would be `-lmyLibrary`.

### Statically-linked `main`

```console
cppbuild/build> clang++ -o mainStatic $projectDir/src/main.cpp -I$projectDir/src
-L$projectDir/build -lcppbuild
cppbuild/build> ./mainStatic
Success!
```

### Dynamically-linked `main` (shared library)

```console
cppbuild/build> clang++ -o mainShared $projectDir/src/main.cpp -I$projectDir/src
-L$projectDir/build/so -lcppbuild
cppbuild/build> ./mainShared
Success!
```

When all is said and done, here is the final project directory structure:

```
├── build
│   ├── add.o
│   ├── libcppbuild.a
│   ├── mainShared
│   ├── mainStatic
│   ├── so
│   │   ├── add.o
│   │   ├── libcppbuild.so
│   │   └── subtract.o
│   └── subtract.o
└── src
    ├── add.cpp
    ├── add.hpp
    ├── main.cpp
    ├── subtract.cpp
    └── subtract.hpp
```

## Conclusion 

And there we have it: we've manually compiled a C++ program that uses an external
library. The important parts were:

- Specifying the directory of the headers
- Specifying the directory storing the external binaries
- Deciding to statically link or dynamically link.
- Linking to the correct binaries.

The build process is much more complex for larger projects due to the many compiler
customization flags, but the core is the same: what do we include, where do we find the
declarations (headers), where do we find the implementations (binaries), and which binaries
do we use? These questions comprised the bulk of my confusion when branching further than
toy programs that only used the standard library. System
libraries are handled automatically, a fact that rendered me helpless when I needed to link a
third-pary library on my own. In practice, CMake handles this for us, but it is imperative
to know *how* such a thing is done.
