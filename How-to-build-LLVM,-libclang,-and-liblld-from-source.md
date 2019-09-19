## Windows

### Pre-built Binaries

Alternately use the [pre-built binaries](https://github.com/zig-lang/zig/wiki/Building-Zig-on-Windows).

### Setup

[Download llvm and cfe](http://releases.llvm.org/download.html#9.0.0) and unzip each to their own directory. Ensure no directories have spaces in them. For example:

 * `C:\Users\Andy\llvm-9.0.0.src`
 * `C:\Users\Andy\cfe-9.0.0.src`

Install [Build Tools for Visual Studio 2019](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2019).

Install [Python 2.7](https://www.python.org).

Install [CMake](http://cmake.org).

### LLVM

Using the start menu, run **x64 Native Tools Command Prompt for VS 2019** and execute these commands, replacing `C:\Users\Andy` with the correct value.

#### Release Mode

```
> mkdir C:\Users\Andy\llvm-9.0.0.src\build-release
> cd C:\Users\Andy\llvm-9.0.0.src\build-release
> "c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_INSTALL_PREFIX=C:\Users\Andy\llvm+clang-9.0.0-win64-msvc-release -DCMAKE_PREFIX_PATH=C:\Users\Andy\llvm+clang-9.0.0-win64-msvc-release -DCMAKE_BUILD_TYPE=Release -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD="AVR" -DLLVM_ENABLE_LIBXML2=OFF
> msbuild /m -p:Configuration=Release INSTALL.vcxproj
```

#### Debug Mode

```
> mkdir C:\Users\Andy\llvm-9.0.0.src\build-debug
> cd C:\Users\Andy\llvm-9.0.0.src\build-debug
> "c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_INSTALL_PREFIX=C:\Users\andy\llvm+clang-9.0.0-win64-msvc-debug -DCMAKE_PREFIX_PATH=C:\Users\andy\llvm+clang-9.0.0-win64-msvc-debug -DCMAKE_BUILD_TYPE=Debug -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD="AVR" -DLLVM_ENABLE_LIBXML2=OFF
> msbuild /m INSTALL.vcxproj
```

### Clang

Using the start menu, run **x64 Native Tools Command Prompt for VS 2019** and execute these commands, replacing `C:\Users\Andy` with the correct value.

#### Release Mode

```
> mkdir C:\Users\Andy\cfe-9.0.0.src\build-release
> cd C:\Users\Andy\cfe-9.0.0.src\build-release
> "c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_INSTALL_PREFIX=C:\Users\Andy\llvm+clang-9.0.0-win64-msvc-release -DCMAKE_PREFIX_PATH=C:\Users\Andy\llvm+clang-9.0.0-win64-msvc-release -DCMAKE_BUILD_TYPE=Release
> msbuild /m -p:Configuration=Release INSTALL.vcxproj
```

#### Debug Mode

```
> mkdir C:\Users\Andy\cfe-9.0.0.src\build-debug
> cd C:\Users\Andy\cfe-9.0.0.src\build-debug
> "c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_INSTALL_PREFIX=C:\Users\andy\llvm+clang-9.0.0-win64-msvc-debug -DCMAKE_PREFIX_PATH=C:\Users\andy\llvm+clang-9.0.0-win64-msvc-debug -DCMAKE_BUILD_TYPE=Debug
> msbuild /m INSTALL.vcxproj
```

## Posix

Typically I use the path `~/local` since it does not require root to install, and it's sandboxed away from the rest of my system. If there's garbage in that directory then I just wipe it and start over.

```
$ cd llvm-9.0.0.src/
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local -DCMAKE_BUILD_TYPE=Release -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD="AVR" -DLLVM_ENABLE_LIBXML2=OFF
$ make install
```

```
$ cd cfe-9.0.0.src/
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local -DCMAKE_BUILD_TYPE=Release
$ make install
```

Then add to your zig cmake line that you got from the readme:
`-DCMAKE_PREFIX_PATH=$HOME/local`

If you get stuck you can look at the CI testing scripts for inspiration, keeping in mind that your environment might be different: https://github.com/zig-lang/zig/tree/master/ci
