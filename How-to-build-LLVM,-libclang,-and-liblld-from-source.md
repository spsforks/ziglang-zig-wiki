## Windows

### Alternate Options to Building from Source

 * [pre-built binaries](https://github.com/ziglang/zig/wiki/Building-Zig-on-Windows#option-2-using-cmake-and-microsoft-visual-studio)
 * [cross-compile using the bootstrap repository](https://github.com/zig-lang/zig-bootstrap)

### Setup

Install [CMake](https://cmake.org/), version 3.15.3 or newer.

[Download llvm, clang, and lld](http://releases.llvm.org/download.html#11.0.0) and unzip each to their own directory. Ensure no directories have spaces in them. For example:

 * `C:\Users\Andy\llvm-11.0.0.src`
 * `C:\Users\Andy\clang-11.0.0.src`
 * `C:\Users\Andy\lld-11.0.0.src`

Install [Build Tools for Visual Studio 2019](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2019). Be sure to select "C++ build tools" when prompted.
 * You must additionally check the optional component labeled **C++ ATL for v142 build tools**.
 * Full list of supported MSVC versions:
   - 2017 (version 15.8)
   - 2019 (version 16)

Install [Python 2.7](https://www.python.org).

Install [CMake 3.17](http://cmake.org). 

### LLVM

Using the start menu, run **x64 Native Tools Command Prompt for VS 2019** and execute these commands, replacing `C:\Users\Andy` with the correct value.

#### Release Mode

```
mkdir C:\Users\Andy\llvm-11.0.0.src\build-release
cd C:\Users\Andy\llvm-11.0.0.src\build-release
"c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_INSTALL_PREFIX=C:\Users\Andy\llvm+clang+lld-11.0.0-x86_64-windows-msvc-release-mt -DCMAKE_PREFIX_PATH=C:\Users\Andy\llvm+clang+lld-11.0.0-x86_64-windows-msvc-release-mt -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_LIBXML2=OFF -DLLVM_USE_CRT_RELEASE=MT
msbuild /m -p:Configuration=Release INSTALL.vcxproj
```

#### Debug Mode

```
mkdir C:\Users\Andy\llvm-11.0.0.src\build-debug
cd C:\Users\Andy\llvm-11.0.0.src\build-debug
"c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_INSTALL_PREFIX=C:\Users\andy\llvm+clang+lld-11.0.0-x86_64-windows-msvc-debug -DCMAKE_PREFIX_PATH=C:\Users\andy\llvm+clang+lld-11.0.0-x86_64-windows-msvc-debug -DCMAKE_BUILD_TYPE=Debug -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD="AVR" -DLLVM_ENABLE_LIBXML2=OFF -DLLVM_USE_CRT_DEBUG=MTd
msbuild /m INSTALL.vcxproj
```

### LLD

Using the start menu, run **x64 Native Tools Command Prompt for VS 2019** and execute these commands, replacing `C:\Users\Andy` with the correct value.

#### Release Mode

```
mkdir C:\Users\Andy\lld-11.0.0.src\build-release
cd C:\Users\Andy\lld-11.0.0.src\build-release
"c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_INSTALL_PREFIX=C:\Users\Andy\llvm+clang+lld-11.0.0-x86_64-windows-msvc-release-mt -DCMAKE_PREFIX_PATH=C:\Users\Andy\llvm+clang+lld-11.0.0-x86_64-windows-msvc-release-mt -DCMAKE_BUILD_TYPE=Release -DLLVM_USE_CRT_RELEASE=MT
msbuild /m -p:Configuration=Release INSTALL.vcxproj
```

#### Debug Mode

```
mkdir C:\Users\Andy\lld-11.0.0.src\build-debug
cd C:\Users\Andy\lld-11.0.0.src\build-debug
"c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_INSTALL_PREFIX=C:\Users\andy\llvm+clang+lld-11.0.0-x86_64-windows-msvc-debug -DCMAKE_PREFIX_PATH=C:\Users\andy\llvm+clang+lld-11.0.0-x86_64-windows-msvc-debug -DCMAKE_BUILD_TYPE=Debug -DLLVM_USE_CRT_DEBUG=MTd
msbuild /m INSTALL.vcxproj
```

### Clang

Using the start menu, run **x64 Native Tools Command Prompt for VS 2019** and execute these commands, replacing `C:\Users\Andy` with the correct value.

#### Release Mode

```
mkdir C:\Users\Andy\clang-11.0.0.src\build-release
cd C:\Users\Andy\clang-11.0.0.src\build-release
"c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_INSTALL_PREFIX=C:\Users\Andy\llvm+clang+lld-11.0.0-x86_64-windows-msvc-release-mt -DCMAKE_PREFIX_PATH=C:\Users\Andy\llvm+clang+lld-11.0.0-x86_64-windows-msvc-release-mt -DCMAKE_BUILD_TYPE=Release -DLLVM_USE_CRT_RELEASE=MT
msbuild /m -p:Configuration=Release INSTALL.vcxproj
```

#### Debug Mode

```
mkdir C:\Users\Andy\clang-11.0.0.src\build-debug
cd C:\Users\Andy\clang-11.0.0.src\build-debug
"c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_INSTALL_PREFIX=C:\Users\andy\llvm+clang+lld-11.0.0-x86_64-windows-msvc-debug -DCMAKE_PREFIX_PATH=C:\Users\andy\llvm+clang+lld-11.0.0-x86_64-windows-msvc-debug -DCMAKE_BUILD_TYPE=Debug -DLLVM_USE_CRT_DEBUG=MTd
msbuild /m INSTALL.vcxproj
```

## Posix

Typically I use the path `~/local` since it does not require root to install, and it's sandboxed away from the rest of my system. If there's garbage in that directory then I just wipe it and start over.

```
cd llvm-11.0.0.src/
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_LIBXML2=OFF
make install
```

```
cd lld-11.0.0.src/
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local -DCMAKE_BUILD_TYPE=Release
make install
```

```
cd clang-11.0.0.src/
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local -DCMAKE_BUILD_TYPE=Release
make install
```

Then add to your zig cmake line that you got from the readme:
`-DCMAKE_PREFIX_PATH=$HOME/local`

If you get stuck you can look at the CI testing scripts for inspiration, keeping in mind that your environment might be different: https://github.com/zig-lang/zig/tree/master/ci
