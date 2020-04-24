# Compiling and Running

_Note to editors: Please do not duplicate information on this page in the specific guides. Where information is already given in multiple guides consider moving it to this page instead._

This generic guide covers the steps to build MuseScore that are the same on all platforms. See the platform-specific guides for details that only apply to your platform.

* [Windows](WindowsCompiling.md)
* [Mac](MacCompiling.md)
* [Linux & BSD](LinuxCompiling.md)

## Prerequisites

Install Git and follow the steps in [Git Workflow](../WorkflowAndGuidelines/GitWorkflow.md) to get MuseScore's code repository on your machine.

Install CMake, Qt, and the other dependencies (or you can try to compile without them and then install them as needed based on error messages you see in the terminal).

All programs must be installed to a location mentioned in your PATH environment variable.

## Basic steps

Once you have MuseScore's code and dependencies on your machine, the steps to compile are the same as for any CMake project. The following commands work on all platforms (Windows, macOS, Linux) and in all shells (Bash, PowerShell, CMD, etc.). Subsequent sections of this guide go into more detail about what the commands do.

```bash
mkdir my_build_dir                              # 1. Create build directory
cd my_build_dir
cmake .. -DCMAKE_INSTALL_PREFIX=my_install_dir  # 2. Configure (Mac: add `-GXCode`)
cmake --build .                                 # 3. Build
cmake --build . --target install                # 4. Install (optional)
```

These commands are likely to fail the first time you try. Pay attention to error messages displayed in the terminal output as they give clues about how to proceed.

_Note to developers: Please ensure that error messages include information about what the problem is and also how to fix it. This reduces the need for people to rely on a written tutorial._

## 1. Create build directory

```bash
mkdir my_build_dir
cd my_build_dir
```

The build directory is where compiled code will go. You can name the build directory whatever you like.

Tip: You can create multiple build directories for different configurations of the project (e.g. different platforms, compilers, git branches, or build types such as Release and Debug). This saves having to do a clean build every time you switch configuration.

## 2. Configure

```bash
cmake .. -G[generator] -D[option]=[value]  # configure
```

During the configure step, CMake reads build rules from the top-level CMakeLists.txt and uses them to generate a native build system. This is just a new set of build rules that will work with whichever operating system, processor architecture, compiler toolchains, and IDE you happen to be using.

Note: Build rules are also taken from other CMakeLists.txt and *.cmake files in subdirectories mentioned in the top-level file.

### CMake Generators

By default, CMake tries to guess which native tool you want to build with. If you want it to generate code for a different tool then you can tell it to do so using the `-G[generator]` option at the configure stage. On macOS, only the XCode generator is currently supported, so you must use it as follows:

``` bash
cmake .. -GXCode -D[option]=[value]  # configure for XCode (only on macOS)
```

On other platforms you can use any available generator, or you can leave out the -G option to make CMake use the default one.

The CMake manual has a [list of all generators](https://cmake.org/cmake/help/latest/manual/cmake-generators.7.html) and `cmake --help` will show the ones that are available on your system.

### CMake Variables

You can pass options and variables to the configure step using the `-D[variable]=[value]` syntax. For example:

```bash
cmake .. -DCMAKE_INSTALL_PREFIX=my_install_dir  # configure with variable
```

This sets the CMake variable [CMAKE_INSTALL_PREFIX](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX.html) to the value `my_install_dir`, which means that during the install step the compiled binary and accompanying resources (fonts, translations, templates, etc.) will be copied to a directory called `my_install_dir` inside your build directory.

### Built-in variables

CMake's online documentation has a [complete list of built-in variables](https://cmake.org/cmake/help/latest/manual/cmake-variables.7.html). Here are some of the key ones:

* [CMAKE_INSTALL_PREFIX](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX.html) - directory to install the compiled program and accompanying resources.
  * The default locations are `/usr/local` on UNIX and `C:/Program Files/${PROJECT_NAME}` on Windows, but it's a good idea to use a relative path instead as this avoids the need for root/admin privileges during the install step.
* [CMAKE_BUILD_TYPE](https://cmake.org/cmake/help/latest/variable/CMAKE_BUILD_TYPE.html) - whether to create a Release or a Debug build.
  * Developers usually set this to `Debug` because `Release` discards symbols necessary for debugging. Windows users might prefer `RelWithDebug` as this enables optimisations that increase performance while still retaining debug symbols.
* [CMAKE_MAKE_PROGRAM](https://cmake.org/cmake/help/latest/variable/CMAKE_MAKE_PROGRAM.html) - full path to native build tool (or just the tool's name if it's in `PATH`).
  * You don't have to set this unless the tool has a non-standard name or location. MinGW users must set it to `mingw32-make.exe`.

### Project variables

MuseScore defines it's own options and variables mostly in the [top-level CMakeLists.txt](https://github.com/musescore/MuseScore/blob/master/CMakeLists.txt). Examples:

* `BUILD_64` - Enable 64 bit builds.
  * Default is ON. Has no effect except with MSVC on Windows.
* `MSCORE_INSTALL_SUFFIX` - string to append to the name of the `mscore` binary to prevent conflicts.
  * Has no effect except on Linux where `MSCORE_INSTALL_SUFFIX=foo` would give `mscorefoo`.
  * Default value is "" (i.e. the empty string) so binary is just called `mscore`.
  * Use values that start with -portable to enable AppImage builds on Linux.

Project variables are set on the command line with the `-D` option in the same way as other CMake variables:

```bash
cmake .. -DBUILD_64=OFF  # configure with option
```

If you need to call CMake with lots of options then you could create a shell script or batch file to make it easy to run the same commands again in the future. You may notice that there are some scripts in [repository root folder](https://github.com/musescore/MuseScore) that do this already (see the Makefiles and msvc_build.bat). These are provided as a convenience, but it is better to avoid them and write your own if you can. CMake's purpose is to abstract the build process so that it is the same on all platforms. Providing a separate script for each platform undermines this purpose and discourages developers from learning how to use CMake properly.