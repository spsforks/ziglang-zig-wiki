## Windows

### Pre-built Binaries

Alternately use the [pre-built binaries](https://github.com/zig-lang/zig/wiki/Building-Zig-on-Windows).

### Setup

[Download llvm and cfe](http://releases.llvm.org/download.html#7.0.0) and unzip each to their own directory. Ensure no directories have spaces in them. For example:

 * `C:\Users\Andy\llvm-7.0.0.src`
 * `C:\Users\Andy\cfe-7.0.0.src`

Install [Visual Studio Community 2017 (version 15.8)](https://my.visualstudio.com/Downloads?q=visual%20studio%202017%2015.8&wt.mc_id=o~msft~vscom~older-downloads).

Install [Python 2.7](https://www.python.org).

Install [CMake](http://cmake.org).

### LLVM

Using the start menu, run **VS2015 x64 Native Tools Command Prompt** and execute these commands, replacing `C:\Users\Andy` with the correct value.

#### Release Mode

```
> mkdir C:\Users\Andy\llvm-7.0.0.src\build-release
> cd C:\Users\Andy\llvm-7.0.0.src\build-release
> "c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G"Visual Studio 15 2017 Win64" -DCMAKE_INSTALL_PREFIX=C:\Users\Andy\llvm+clang-7.0.0-win64-msvc-release -DCMAKE_PREFIX_PATH=C:\Users\Andy\llvm+clang-7.0.0-win64-msvc-release -DCMAKE_BUILD_TYPE=Release
> msbuild -p:Configuration=Release INSTALL.vcxproj
```

#### Debug Mode

```
> mkdir C:\Users\Andy\llvm-7.0.0.src\build-debug
> cd C:\Users\Andy\llvm-7.0.0.src\build-debug
> "c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G"Visual Studio 15 2017 Win64" -DCMAKE_INSTALL_PREFIX=C:\Users\andy\llvm+clang-7.0.0-win64-msvc-debug -DCMAKE_PREFIX_PATH=C:\Users\andy\llvm+clang-7.0.0-win64-msvc-debug -DCMAKE_BUILD_TYPE=Release
> msbuild INSTALL.vcxproj
```

### Clang

Using the start menu, run **VS2015 x64 Native Tools Command Prompt** and execute these commands, replacing `C:\Users\Andy` with the correct value.

#### Release Mode

```
> mkdir C:\Users\Andy\cfe-7.0.0.src\build-release
> cd C:\Users\Andy\cfe-7.0.0.src\build-release
> "c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G"Visual Studio 15 2017 Win64" -DCMAKE_INSTALL_PREFIX=C:\Users\Andy\llvm+clang-7.0.0-win64-msvc-release -DCMAKE_PREFIX_PATH=C:\Users\Andy\llvm+clang-7.0.0-win64-msvc-release -DCMAKE_BUILD_TYPE=Release
> msbuild -p:Configuration=Release INSTALL.vcxproj
```

#### Debug Mode

```
> mkdir C:\Users\Andy\cfe-7.0.0.src\build-debug
> cd C:\Users\Andy\cfe-7.0.0.src\build-debug
> "c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G"Visual Studio 15 2017 Win64" -DCMAKE_INSTALL_PREFIX=C:\Users\andy\llvm+clang-7.0.0-win64-msvc-debug -DCMAKE_PREFIX_PATH=C:\Users\andy\llvm+clang-7.0.0-win64-msvc-debug -DCMAKE_BUILD_TYPE=Release
> msbuild INSTALL.vcxproj
```

## Posix

Typically I use the path `~/local` since it does not require root to install, and it's sandboxed away from the rest of my system. If there's garbage in that directory then I just wipe it and start over.

```
$ cd llvm-7.0.0.src/
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local -DCMAKE_BUILD_TYPE=Release
$ make install
```

```
$ cd cfe-7.0.0.src/
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local -DCMAKE_BUILD_TYPE=Release
$ make install
```

Then add to your zig cmake line that you got from the readme:
`-DCMAKE_PREFIX_PATH=$HOME/local`

If you get stuck you can look at the CI testing scripts for inspiration, keeping in mind that your environment might be different: https://github.com/zig-lang/zig/tree/master/ci
