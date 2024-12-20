# Installing Qt without a Qt Account: A Frustrating Journey

**Note**: If you're reading this and just want to know how to install Qt without needing an account, read [the corresponding article](install-no-account.md). This post is more of a vent/reflection on the installation process.

---

As part of a project to improve my C++, I need to create a GUI. From my research, `Qt` is the *de facto* framework for creating user applications using C++, and it has the additional benefit that it's free, open source software (FOSS). However, I learned through a three-day trial-and-error process that some things that are free are paid for with time.

## Attempt 1: `qt.io`

Most of my experience with FOSS has been smooth sailing: just find the package via the language's package manager. If using a language like C++ that has a more...distributed ecosystem, install the package through a pre-compiled binary, or build it directly from source. Piece of cake. While The Qt Company is generous enough to offer a full online installer that is allegedly simple to use (I wouldn't know; I refused to use it), the company also requires you to register an account in order to use it.
The Qt Company likely has some logical reason for adopting this practice, but, emotionally, I was left with a soured first impression. In any case, I looked to other options for installing the framework.

## Attempt 2: `vcpkg`

Being relatively new to C++, I had some trepidation about building from source. I knew that I didn't want to do a system-wide installation, but I wasn't sure how to link a local installation to my tools (clangd, CMake, compilers, etc.). Since my project has multiple dependencies and package management is a recommended practice among experienced developers anyway, I decided I'd attempt to use a package manager before trying a manual build. 

I'm unfamiliar with the merits of vcpkg vs. Conan, so I picked the former because it only required CMake. I later learned that Python was needed anyway as a result of a Jinja2 dependency, so I'm not sure if it really matters which system to use in the end.

Ultimately, the `vcpkg` attempt didn't work. I ran into a lot of build failures due to missing system dependencies, but I resolved those by installing `bison` and `xorg-dev` packages on my Ubuntu 20.04 virtual machine. The real show-stopper was LLVM. Being one of Qt's dependencies, `vcpkg` attempted to install and build the LLVM suite. I eventually ran out of disk space on my 100GB VM due to build artifacts (the buildtree was 87GB at that point). Being naive, I'd created that VM with a fixed size, so I had to delete it and make a new one with dynamic size, this time with 250GB. 

The increased disk space and specifying the `--clean-after-build` option on `vcpkg install` worked, but then the build failed anyway due to insufficient memory—memory that I assume to be RAM-based. Online searching revealed that this is not an uncommon issue with building LLVM,and some users even have this issue occur with large amounts of RAM (ex. 32GB). I don't have the technical expertise to troubleshoot the issue on my own, so I had to abandon this method. I do hope to use `vcpkg` or `Conan` in the future, however, because the concept of package management in a distributed ecosystem like C++ is fascinating. With this unfortunate failure, I moved on to my next attempt.

## Attempt 3: Building from Source

This attempt actually produced a working library. I followed the instructions on [building Qt from Git](https://wiki.qt.io/Building_Qt_5_from_Git) using [`qtbase`](https://github.com/qt/qtbase) instead of the mega-package. I also learned to both link a local library installation to my development tools and to compile a Qt program using CMake (and qmake as an alternative). The problem with this attempt, however, was that most resources about Qt that I found use `QtQML` instead of `QtWidgets`, and my installation didn't have the former. I tried to build the mega-package, but [LLVM James blocked me again](https://www.youtube.com/watch?v=-zd62MxKXp8). So, while this method worked initially, it ultimately failed for my potential use-case.

## Attempt 4: apt install

Long story short, this one had the same issue as [Attempt 3](#attempt-3:-building-from-source).

## Attempt 5: `aqtinstall`

By now, I was at my wit's end, but I stumbled across a [Reddit comment](https://old.reddit.com/r/QtFramework/comments/ms7hf6/how_do_i_install_qt5_without_an_account/guqwyhx/) about a third-party tool called `aqt`. This, it turns out, was the answer. Because Ubuntu 20.04 has an incompatible Python version, I built Python 3.12 from source, created a virtual environment, and ran `pip install aqtinstall`. The tool downloaded my desired Qt (*with* QML), and I was able to link the installation to my tools. I even wrote a few small practice programs that performed successfully.


## Conclusion

Installing one library/framework shouldn't take multiple days. Part of it was a skill issue, and I did actually learn a lot in the process, but my goodness. If you want to install Qt5 or Qt6 without an account, just use `aqt`.

Now, to be fair, much of my trouble was self-inflicted because I could have just created a Qt account. However, I don't like that requirement, so I chose not to use the installer. In fact, I'm actually conflicted about it overall. For one, I understand wanting to get potential leads on commericial license sales by collecting email addresses. I'm all for generating income from the hard work it takes to produce, maintain, and improve a successful product. However, I don't like feeling like I'm all-but-forced to provide personal information for convenient installation, and I also don't like that this practice is accepted for this specific framework when it almost certainly wouldn't be for other libraries or even Linux itself. Is it still free if I have to exchange my personal information to get it?

Nevertheless, that was my experience with installing Qt, and I sure hope it's easier to learn and use than it was to install.
