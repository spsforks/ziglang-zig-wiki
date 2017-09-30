## Windows

### Pre-built Binaries

Alternately use the [pre-built binaries](https://github.com/zig-lang/zig/wiki/Building-Zig-on-Windows).

### Setup

[Download llvm, cfe, and LLD](http://releases.llvm.org/download.html#5.0.0) and unzip each to their own directory. Ensure no directories have spaces in them. For example:

 * `C:\Users\Andy\llvm-5.0.0.src`
 * `C:\Users\Andy\cfe-5.0.0.src`
 * `C:\Users\Andy\lld-5.0.0.src`

Install [Visual Studio Community 2015 with Update 3](https://my.visualstudio.com/Downloads?q=visual%20studio%202015&wt.mc_id=o~msft~vscom~older-downloads).

Install [Python 2.7](https://www.python.org).

Install [CMake](http://cmake.org).

### LLVM

Run the CMake GUI.

**Browse Source...** and select `C:\Users\Andy\llvm-5.0.0.src`.

**Browse Build...** and select `C:\Users\Andy\llvm-5.0.0.src\build`.

Click **Add Entry** and add these entries:

 * `CMAKE_BUILD_TYPE` type `STRING` value `Release`.
 * `CMAKE_PREFIX_PATH` type `PATH` value `C:\Users\Andy\llvm+clang+lld-5.0.0-win64-msvc`.
 * `CMAKE_INSTALL_PREFIX` type `PATH` value `C:\Users\Andy\llvm+clang+lld-5.0.0-win64-msvc`.

Click **Configure** and use these options:

 * Specify the generator for this project: `Visual Studio 14 2015 Win64`
 * Optional toolset to use: `host=x64`
 * `Use the default native compilers`

Click **Generate**.

Using the start menu, run **VS2015 x64 Native Tools Command Prompt** and execute these commands:

```
cd C:\Users\Andy\llvm-5.0.0.src\build
msbuild INSTALL.vcxproj
```

### Clang

Run the CMake GUI.

**Browse Source...** and select `C:\Users\Andy\cfe-5.0.0.src`.

**Browse Build...** and select `C:\Users\Andy\cfe-5.0.0.src\build`.

Click **Add Entry** and add these entries:

 * `CMAKE_BUILD_TYPE` type `STRING` value `Release`.
 * `CMAKE_PREFIX_PATH` type `PATH` value `C:\Users\Andy\llvm+clang+lld-5.0.0-win64-msvc`.
 * `CMAKE_INSTALL_PREFIX` type `PATH` value `C:\Users\Andy\llvm+clang+lld-5.0.0-win64-msvc`.

Click **Configure** and use these options:

 * Specify the generator for this project: `Visual Studio 14 2015 Win64`
 * Optional toolset to use: `host=x64`
 * `Use the default native compilers`

Click **Generate**.

Using the start menu, run **VS2015 x64 Native Tools Command Prompt** and execute these commands:

```
cd C:\Users\Andy\cfe-5.0.0.src\build
msbuild INSTALL.vcxproj
```

### LLD

Run the CMake GUI.

**Browse Source...** and select `C:\Users\Andy\lld-5.0.0.src`.

**Browse Build...** and select `C:\Users\Andy\lld-5.0.0.src\build`.

Click **Add Entry** and add these entries:

 * `CMAKE_BUILD_TYPE` type `STRING` value `Release`.
 * `CMAKE_PREFIX_PATH` type `PATH` value `C:\Users\Andy\llvm+clang+lld-5.0.0-win64-msvc`.
 * `CMAKE_INSTALL_PREFIX` type `PATH` value `C:\Users\Andy\llvm+clang+lld-5.0.0-win64-msvc`.

Click **Configure** and use these options:

 * Specify the generator for this project: `Visual Studio 14 2015 Win64`
 * Optional toolset to use: `host=x64`
 * `Use the default native compilers`

Click **Generate**.

Using the start menu, run **VS2015 x64 Native Tools Command Prompt** and execute these commands:

```
cd C:\Users\Andy\lld-5.0.0.src\build
msbuild INSTALL.vcxproj
```

## Posix

Typically I use the path `~/local` since it does not require root to install, and it's sandboxed away from the rest of my system. If there's garbage in that directory then I just wipe it and start over.

```
$ cd llvm-5.0.0.src/
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local -DCMAKE_BUILD_TYPE=Release
$ make install
```

```
$ cd cfe-5.0.0.src/
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local -DCMAKE_BUILD_TYPE=Release
$ make install
```

```
$ cd lld-5.0.0.src/
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local -DCMAKE_BUILD_TYPE=Release
$ make install
```

Then add to your zig cmake line that you got from the readme:
`-DCMAKE_PREFIX_PATH=$HOME/local`

If you get stuck you can look at the CI testing scripts for inspiration, keeping in mind that your environment might be different: https://github.com/zig-lang/zig/tree/master/ci

#### Preferring the local, instead of system llvm

_The following should no longer be required for the latest commit but is left for reference._

If you have a system llvm install then it _may_ possibly be picked up by CMake instead of your local install. Only perform these steps if zig fails to compile at the linking step.

Modify the `cmake/Findllvm.cmake` as follows:

```
# Find the correct local llvm-config executable.
find_program(LLVM_CONFIG_EXE
    NAMES llvm-config5.0 llvm-config
    PATHS
        "~/local/bin"
        "/mingw64/bin"
        "/c/msys64/mingw64/bin"
        "c:/msys64/mingw64/bin"
        "C:/Libraries/llvm-5.0.0/bin"
    NO_DEFAULT_PATH)
```

```
# Do not lookup the local llvm library
#find_library(LLVM_LIBRARY NAMES LLVM)

set(LLVM_LIBRARIES ${LLVM_LIBRARIES} ${LLVM_SYSTEM_LIBS})

#if(LLVM_LIBRARY)
#  set(LLVM_LIBRARIES ${LLVM_LIBRARY})
#endif()
```

Remember to rerun the cmake command and run make again.