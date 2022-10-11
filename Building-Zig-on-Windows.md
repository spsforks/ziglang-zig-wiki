**Note**: If you want the latest build of Zig on Windows, you can get it from [the download page](https://ziglang.org/download/).

***

## Option 1: Use the Windows Zig Compiler Dev Kit

This one has the benefit that LLVM, LLD, and Clang are built in Release mode, while your Zig build has the option to be a Debug build. It also works completely independently from MSVC so you don't need it to be installed.

* [zig+llvm+lld+clang-x86_64-windows-gnu-0.10.0-dev.4193+f3a1b5c48.zip](https://ziglang.org/deps/zig+llvm+lld+clang-x86_64-windows-gnu-0.10.0-dev.4193+f3a1b5c48.zip) (168 MiB) (sha256 94583771635593f38be11d55a083eb4bee81ddfca3cb5a7761e64e0cc2516ad3)

If this does not work, the URL for the archive the current master build uses can be found in the [pipelines.yml](https://github.com/ziglang/zig/blob/master/ci/azure/pipelines.yml#L29-L31) file.

Please consider [sponsoring Zig](https://github.com/sponsors/ziglang). ❤️ 

This zip file contains:

 * An older Zig installation.
 * LLVM, LLD, and Clang libraries (.lib and .h files), version 14.0.6, built in Release mode.

Unzip the dev kit and then in cmd.exe in your Zig source checkout:

```bat
$DEVKIT\bin\zig.exe build -p stage3 --search-prefix $DEVKIT --zig-lib-dir lib -Denable-stage1 -Dstatic-llvm -Duse-zig-libcxx -Dtarget=x86_64-windows-gnu
```

Replace `$DEVKIT` with the path to the folder that you unzipped after downloading it from the link above.

Append `-Drelease` for a Release build.

**If you get an error building at this step**, it is most likely that the Zig installation inside the dev kit is too old, and the dev kit needs to be updated. In this case one more step is required:

 1. [Download the latest master branch zip file](https://ziglang.org/download/#release-master).
 2. Unzip, and try the above command again, replacing the path to zig.exe with the path to the zig.exe you just extracted, and also replace the lib\zig folder with the new contents.

You now have the `zig.exe` binary at `stage3\bin\zig.exe` and you can run the tests:

```bat
stage3\bin\zig.exe build test
```

This can take a long time. For tips & tricks on using the test suite, see [Contributing](https://github.com/ziglang/zig/blob/master/.github/CONTRIBUTING.md#editing-source-code).

## Option 2: Using CMake and Microsoft Visual Studio

This one has the benefit that changes to the language or build system won't break your dev kit. This option can be used to upgrade a dev kit.

First, [build LLVM, LLD, and Clang from source using CMake and Microsoft Visual Studio](https://github.com/ziglang/zig/wiki/How-to-build-LLVM,-libclang,-and-liblld-from-source#windows). Or, skip this step using this pre-built binary tarball:

* [llvm+clang+lld-13.0.1-x86_64-windows-msvc-release-mt.tar.xz](https://ziglang.org/deps/llvm%2bclang%2blld-13.0.1-x86_64-windows-msvc-release-mt.tar.xz) (432 MiB) (sha256 0abecd9bdf630f8c0f99940752f438cd10dfe4cb35fa1bba054d89e7219fd2ed)

Please consider [sponsoring Zig](https://github.com/sponsors/ziglang). ❤️ 

Unzip this file to a directory with no spaces, such as `C:\Users\Andy\`. It contains a single directory, so when you do this the full path will be e.g. `C:\Users\Andy\llvm+clang+lld-13.0.1-x86_64-windows-msvc-release-mt\`.

Install [Build Tools for Visual Studio 2019](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2019). Be sure to select "Desktop development with C++" when prompted.
 * You must additionally check the optional component labeled **C++ ATL for v142 build tools**.

Install [CMake](http://cmake.org).

Use [git](https://git-scm.com/) to clone the zig repository to a path with no spaces, e.g. `C:\Users\Andy\zig`.

Using the start menu, run **x64 Native Tools Command Prompt for VS 2019** and execute these commands, replacing `C:\Users\Andy` with the correct value.

```bat
mkdir C:\Users\Andy\zig\build-release
cd C:\Users\Andy\zig\build-release
"c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_PREFIX_PATH=C:\Users\Andy\llvm+clang+lld-13.0.1-x86_64-windows-msvc-release-mt -DCMAKE_BUILD_TYPE=Release
msbuild -p:Configuration=Release INSTALL.vcxproj
```

You now have the `zig.exe` binary at `bin\zig.exe` and you can run the tests:

```bat
bin\zig.exe build test
```

This can take a long time. For tips & tricks on using the test suite, see [Contributing](https://github.com/ziglang/zig/blob/master/.github/CONTRIBUTING.md#editing-source-code).

Note: In case you get the error "llvm-config not found" (or similar), make sure that you have **no** trailing slash (`/` or `\`) at the end of the `-DCMAKE_PREFIX_PATH` value. 
