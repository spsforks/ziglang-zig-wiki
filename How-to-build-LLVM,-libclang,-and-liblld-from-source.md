## Windows

### Alternate Options to Building from Source

 * [pre-built binaries](https://github.com/ziglang/zig/wiki/Building-Zig-on-Windows#option-2-using-cmake-and-microsoft-visual-studio)
 * [cross-compile using the bootstrap repository](https://github.com/ziglang/zig-bootstrap)

### Setup

Install [CMake](https://cmake.org/), version 3.17 or newer.

[Download llvm, clang, and lld](http://releases.llvm.org/download.html#13.0.0) and unzip each to their own directory. Ensure no directories have spaces in them. For example:

 * `C:\Users\Andy\llvm-13.0.0.src`
 * `C:\Users\Andy\clang-13.0.0.src`
 * `C:\Users\Andy\lld-13.0.0.src`

Install [Build Tools for Visual Studio 2019](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2019). Be sure to select "C++ build tools" when prompted.
 * You must additionally check the optional component labeled **C++ ATL for v142 build tools**.
 * Full list of supported MSVC versions:
   - 2017 (version 15.8)
   - 2019 (version 16)

Install [Python 3.9.4](https://www.python.org). Tick the box to add python to your PATH environment variable.

### LLVM

Using the start menu, run **x64 Native Tools Command Prompt for VS 2019** and execute these commands, replacing `C:\Users\Andy` with the correct value.

#### Release Mode

```bat
mkdir C:\Users\Andy\llvm-13.0.0.src\build-release
cd C:\Users\Andy\llvm-13.0.0.src\build-release
"c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_INSTALL_PREFIX=C:\Users\Andy\llvm+clang+lld-13.0.0-x86_64-windows-msvc-release-mt -DCMAKE_PREFIX_PATH=C:\Users\Andy\llvm+clang+lld-13.0.0-x86_64-windows-msvc-release-mt -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_LIBXML2=OFF -DLLVM_USE_CRT_RELEASE=MT
msbuild /m -p:Configuration=Release INSTALL.vcxproj
```

#### Debug Mode

```bat
mkdir C:\Users\Andy\llvm-13.0.0.src\build-debug
cd C:\Users\Andy\llvm-13.0.0.src\build-debug
"c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_INSTALL_PREFIX=C:\Users\andy\llvm+clang+lld-13.0.0-x86_64-windows-msvc-debug -DCMAKE_PREFIX_PATH=C:\Users\andy\llvm+clang+lld-13.0.0-x86_64-windows-msvc-debug -DCMAKE_BUILD_TYPE=Debug -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD="AVR" -DLLVM_ENABLE_LIBXML2=OFF -DLLVM_USE_CRT_DEBUG=MTd
msbuild /m INSTALL.vcxproj
```

### LLD

Using the start menu, run **x64 Native Tools Command Prompt for VS 2019** and execute these commands, replacing `C:\Users\Andy` with the correct value.

#### Release Mode

```bat
mkdir C:\Users\Andy\lld-13.0.0.src\build-release
cd C:\Users\Andy\lld-13.0.0.src\build-release
"c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_INSTALL_PREFIX=C:\Users\Andy\llvm+clang+lld-13.0.0-x86_64-windows-msvc-release-mt -DCMAKE_PREFIX_PATH=C:\Users\Andy\llvm+clang+lld-13.0.0-x86_64-windows-msvc-release-mt -DCMAKE_BUILD_TYPE=Release -DLLVM_USE_CRT_RELEASE=MT
msbuild /m -p:Configuration=Release INSTALL.vcxproj
```

#### Debug Mode

```bat
mkdir C:\Users\Andy\lld-13.0.0.src\build-debug
cd C:\Users\Andy\lld-13.0.0.src\build-debug
"c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_INSTALL_PREFIX=C:\Users\andy\llvm+clang+lld-13.0.0-x86_64-windows-msvc-debug -DCMAKE_PREFIX_PATH=C:\Users\andy\llvm+clang+lld-13.0.0-x86_64-windows-msvc-debug -DCMAKE_BUILD_TYPE=Debug -DLLVM_USE_CRT_DEBUG=MTd
msbuild /m INSTALL.vcxproj
```

### Clang

Using the start menu, run **x64 Native Tools Command Prompt for VS 2019** and execute these commands, replacing `C:\Users\Andy` with the correct value.

#### Release Mode

```bat
mkdir C:\Users\Andy\clang-13.0.0.src\build-release
cd C:\Users\Andy\clang-13.0.0.src\build-release
"c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_INSTALL_PREFIX=C:\Users\Andy\llvm+clang+lld-13.0.0-x86_64-windows-msvc-release-mt -DCMAKE_PREFIX_PATH=C:\Users\Andy\llvm+clang+lld-13.0.0-x86_64-windows-msvc-release-mt -DCMAKE_BUILD_TYPE=Release -DLLVM_USE_CRT_RELEASE=MT
msbuild /m -p:Configuration=Release INSTALL.vcxproj
```

#### Debug Mode

```bat
mkdir C:\Users\Andy\clang-13.0.0.src\build-debug
cd C:\Users\Andy\clang-13.0.0.src\build-debug
"c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_INSTALL_PREFIX=C:\Users\andy\llvm+clang+lld-13.0.0-x86_64-windows-msvc-debug -DCMAKE_PREFIX_PATH=C:\Users\andy\llvm+clang+lld-13.0.0-x86_64-windows-msvc-debug -DCMAKE_BUILD_TYPE=Debug -DLLVM_USE_CRT_DEBUG=MTd
msbuild /m INSTALL.vcxproj
```

## Posix

This guide will get you both a Debug build of LLVM, and/or a Release build of LLVM.
It intentionally does not require privileged access, using a prefix inside your home
directory instead of a global installation.

### Release

This is the generally recommended approach.

```
cd ~/Downloads
git clone --depth 1 https://github.com/llvm/llvm-project llvm-project-13
cd llvm-project-13
git checkout release/13.x

# LLVM
cd llvm
mkdir build-release
cd build-release
cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local/llvm13-release -DCMAKE_PREFIX_PATH=$HOME/local/llvm13-release -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_LIBXML2=OFF -G Ninja -DLLVM_PARALLEL_LINK_JOBS=1
ninja install
cd ../..

# LLD
cd lld
mkdir build-release
cd build-release
cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local/llvm13-release -DCMAKE_PREFIX_PATH=$HOME/local/llvm13-release -DCMAKE_BUILD_TYPE=Release  -G Ninja -DLLVM_PARALLEL_LINK_JOBS=1 -DCMAKE_CXX_STANDARD=17
ninja install
cd ../..

# Clang
cd clang
mkdir build-release
cd build-release
cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local/llvm13-release -DCMAKE_PREFIX_PATH=$HOME/local/llvm13-release -DCMAKE_BUILD_TYPE=Release  -G Ninja -DLLVM_PARALLEL_LINK_JOBS=1
ninja install
cd ../..
```

### Debug

This is occasionally needed when debugging Zig's LLVM backend.

```
# Skip this step if you already did it for Release above.
cd ~/Downloads
git clone --depth 1 https://github.com/llvm/llvm-project llvm-project-13
cd llvm-project-13
git checkout release/13.x

# LLVM
cd llvm
mkdir build-debug
cd build-debug
cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local/llvm13-debug -DCMAKE_PREFIX_PATH=$HOME/local/llvm13-debug -DCMAKE_BUILD_TYPE=Debug -DLLVM_ENABLE_LIBXML2=OFF -DLLVM_ENABLE_TERMINFO=OFF -G Ninja -DLLVM_PARALLEL_LINK_JOBS=1
ninja install
cd ../..

# LLD
cd lld
mkdir build-debug
cd build-debug
cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local/llvm13-debug -DCMAKE_PREFIX_PATH=$HOME/local/llvm13-debug -DCMAKE_BUILD_TYPE=Release  -G Ninja -DLLVM_PARALLEL_LINK_JOBS=1 -DCMAKE_CXX_STANDARD=17
ninja install
cd ../..

# Clang
cd clang
mkdir build-debug
cd build-debug
cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local/llvm13-debug -DCMAKE_PREFIX_PATH=$HOME/local/llvm13-debug -DCMAKE_BUILD_TYPE=Release  -G Ninja -DLLVM_PARALLEL_LINK_JOBS=1
ninja install
cd ../..
```

Then add to your Zig CMake line that you got from the README.md:
`-DCMAKE_PREFIX_PATH=$HOME/local/llvm13-debug` or `-DCMAKE_PREFIX_PATH=$HOME/local/llvm13-release`
depending on whether you want Debug or Release LLVM.
