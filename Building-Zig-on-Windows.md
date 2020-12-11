**Note**: If you want the latest build of Zig on Windows, you can get it from [the download page](https://ziglang.org/download/).

***

## Option 1: Use the Windows Zig Compiler Dev Kit

This one has the benefit that LLVM, LLD, and Clang are built in Release mode, while your Zig build has the option to be a Debug build. This is not the case when building using MSVC.

* [zig+llvm+lld+clang-x86_64-windows-gnu-0.7.0+8076894d8.zip](https://ziglang.org/deps/zig%2bllvm%2blld%2bclang-x86_64-windows-gnu-0.7.0+8076894d8.zip) (118 MiB) (sha256 06ae35bfac96aa77f8a992db6e2c1abd466daf1167fe05c931a4483922026168)

Please consider [sponsoring Zig](https://github.com/sponsors/ziglang). ❤️ 

This zip file contains:

 * A Zig installation
 * LLVM, LLD, and Clang libraries (.lib and .h files), built in Release mode

With this option, you do not need to install anything else! Seriously! Just unzip it and then in cmd.exe in your Zig source checkout:

```
C:\Users\andy\Downloads\zig+llvm+lld+clang-x86_64-windows-gnu-0.7.0+1afea36a1\bin\zig.exe build -Dstage1 -Dtarget=native-native-gnu --search-prefix C:\Users\andy\Downloads\zig+llvm+lld+clang-x86_64-windows-gnu-0.7.0+1afea36a1
```

Of course, replace `C:\Users\andy\Downloads\` with the path to the directory that contains your unzipped dev kit.

Once [#6565](https://github.com/ziglang/zig/issues/6565) is implemented, the `-Dtarget=native-native-gnu` option will no longer be needed.

You now have the `zig.exe` binary at `zig-cache\bin\zig.exe` and you can run the tests:

```
zig-cache\bin\zig.exe build test
```

This can take a long time. For tips & tricks on using the test suite, see [Contributing](https://github.com/ziglang/zig/blob/master/CONTRIBUTING.md#editing-source-code).

## Option 2: Using CMake and Microsoft Visual Studio

First, [build LLVM, LLD, and Clang from source using CMake and Microsoft Visual Studio](https://github.com/ziglang/zig/wiki/How-to-build-LLVM,-libclang,-and-liblld-from-source#windows). Or, skip this step using this pre-built binary tarball:

* [llvm+clang+lld-11.0.0-x86_64-windows-msvc-release-mt.tar.xz](https://ziglang.org/deps/llvm%2bclang%2blld-11.0.0-x86_64-windows-msvc-release-mt.tar.xz) (474 MiB) (sha256 1dc923ecfc8d0e868eb6a965729318fcebc90c0495db72a310e14a260cd2262a)

Please consider [sponsoring Zig](https://github.com/sponsors/ziglang). ❤️ 

Unzip this file to a directory with no spaces, such as `C:\Users\Andy\`. It contains a single directory, so when you do this the full path will be e.g. `C:\Users\Andy\llvm+clang-11.0.0-win64-msvc-mt\`.

Install [Build Tools for Visual Studio 2019](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2019). Be sure to select "C++ build tools" when prompted.
 * You must additionally check the optional component labeled **C++ ATL for v142 build tools**.

Install [CMake](http://cmake.org).

Use [git](https://git-scm.com/) to clone the zig repository to a path with no spaces, e.g. `C:\Users\Andy\zig`.

Using the start menu, run **x64 Native Tools Command Prompt for VS 2019** and execute these commands, replacing `C:\Users\Andy` with the correct value.

```
mkdir C:\Users\Andy\zig\build-release
cd C:\Users\Andy\zig\build-release
"c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_PREFIX_PATH=C:\Users\Andy\llvm+clang+lld-11.0.0-x86_64-windows-msvc-release-mt -DCMAKE_BUILD_TYPE=Release
msbuild -p:Configuration=Release INSTALL.vcxproj
```

You now have the `zig.exe` binary at `bin\zig.exe` and you can run the tests:

```
bin\zig.exe build test
```

This can take a long time. For tips & tricks on using the test suite, see [Contributing](https://github.com/ziglang/zig/blob/master/CONTRIBUTING.md#editing-source-code).

Note: In case you get the error "llvm-config not found" (or similar), make sure that you have **no** trailing slash (`/` or `\`) at the end of the `-DCMAKE_PREFIX_PATH` value. 
