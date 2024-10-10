# How to Install Qt without an Account

This is an article on installing the Qt framework without signing up for an account or building from source.

## `aqt`

The [Another Qt Install (aqt)](https://github.com/miurahr/aqtinstall) CLI tool is a Qt installer distributed via Python's pip. It requires Python 3.9 or above, so if your system has a version lower than that, I recommend building a compatible Python version from source, creating a virtual environment using the resulting Python build's `venv` module, and running the pip installation command in the designated environment.


Otherwise, this assumes that `aqt` is installed properly and available for use. 

## Installing Qt

`aqt` installs Qt using the Qt distribution site. At the time of writing, I was able to find a wide variety of Qt5 and Qt6 desktop distributions. Read [the documentation](https://aqtinstall.readthedocs.io/en/latest/) in order to install your desired version, documentation, and tools. The instructions are very easy to follow, and the examples are excellent.

As an example, I'll run through what I did. The tool installs into the current working directory, so I first created a designated folder, `Qt`.

```bash
$ source <your python virtual environment>
(qtvenv) $ mkdir Qt
(qtvenv) $ cd Qt
```

To list the versions available, I used the `aqt list-qt` command:

```bash
# in the created Qt folder
(qtvenv) $ aqt list-qt linux desktop 
# ... a lot of Qt versions are output ...
```

Once I confirmed my desired version was available (Qt 5.15.2), I checked to see what architectures were available for Linux:

```bash
(qtvenv) $ aqt list-qt linux desktop --arch 5.15.2
gcc_64
```

From here, install the Qt version for your desired architecture

```bash
(qtvenv) $ aqt install-qt linux desktop 5.15.2 gcc_64
# ... installer runs and outputs logs to console ...
(qtvenv) $ ls
5.15.2/ aqtinstall.log
```

That's it!. Qt is now present on your local system.

## Linking to Tooling

This section is more for me since I want to catalog what I did to make everything work properly. I'm using the following tools:

- Clangd
- CMake
- QtCreator (begrudgingly, but it's a nice IDE).

### clangd

Getting `clangd` to recognize a local installation took some time to figure out, but web searches led me to glory. The two main files in consideration are `.clangd` and `compile_commands.json`—both of which would be placed in a given project's top level.

There are [some differences](https://github.com/clangd/clangd/discussions/1985) between the two files, but `compile_commands.json` is the more important one in this case. It's a generated file and requires the specification of `set(CMAKE_EXPORT_COMPILE_COMMANDS ON)` in `CMakeLists.txt`. The file will be produced in whereever the build directory is specified (`cmake -B <build dir>`). From there, create a symlink to the file in the project root, which I'll refer to as `$project`:

```bash
# in some project directory, $project.
$ ln -s $project/build/compile_commands.json $project
```

Now, `clangd` will see the changes in the build, relieving you from having to manually move the file for each update.

If you are using a `.clangd` file, you need to specify the `CompileFlags` via `Add`, but I think it's still easier to use the generated `compile_commands.json` file by the following:

```
# .clangd
CompileFlags:
    CompilationDatabase:
        build/  # or whatever you call your CMake build directory
```

### CMake

CMake has Qt-specific settings, so follow [the Qt documentation instructions](https://doc.qt.io/qt-6/cmake-manual.html) ([for Qt5](https://doc.qt.io/qt-5/cmake-get-started.html)). The most important part, however, is the setting of the `CMAKE_PREFIX_PATH` variable. This can be done in `cmake -D` invocations or set as an environment variable. If the latter, the command would be

```bash
$ export CMAKE_PREFIX_PATH=<Qt install directory>/<Qt version>/<architecture>
```

So for the version installed above, the command is 

```bash
# call the Qt installation directory $Qt
$ export CMAKE_PREFIX_PATH="$Qt/5.15.2/gcc_64"
```

This will allow CMake to find the Qt installation using the `find_package()` function.

Be sure to list the Qt modules used as arguments to the `COMPONENTS` section of `find_package()`. The linked documentation provides well-written, easy-to-follow examples.

### QtCreator

To tell the QtCreator IDE where the Qt installation is, you [specify the location of the `qmake` executable binary](https://doc.qt.io/qtcreator/creator-project-qmake.html). To do so, go to 'Preferences' (the location of which depends on the version of QtCreator). Under 'Kit', click the 'Qt Versions' tab and click 'Add' under the Manual section. Then, browse to the path of `qmake`. In the context of this article's installation, the path is `$Qt/5.15.2/gcc_64/bin/qmake`.


#### Documentation

If you installed documentation via `aqt install-doc`, link it to QtCreator by following [the instructions](https://doc.qt.io/qtcreator/creator-how-to-add-external-documentation.html) in the documentation. It's really easy to follow and has helpful example images.

## I Installed QtCreator via aqt, how can I run it?

Add the `qtcreator` binary to your `$PATH` by symlinking. For example, if `$HOME/.local/bin` is on your PATH:

```bash
$ ln -s $Qt/Tools/QtCreator/bin/qtcreator $HOME/.local/bin
```

Now, you can run the `qtcreator` command.

Alternatively, you can add the directory `$Qt/Tools/QtCreator/bin` to your $PATH:  `export PATH=<the path>:$PATH`

---

That's all for installing Qt.
