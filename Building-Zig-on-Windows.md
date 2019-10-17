**Note**: If you want the latest build of Zig on Windows, you can get it from [the download page](https://ziglang.org/download/).

***

Here is llvm and clang x64 built with MSVC in release mode:

* [llvm+clang-9.0.0-win64-msvc-mt.tar.xz](https://ziglang.org/deps/llvm%2bclang-9.0.0-win64-msvc-mt.tar.xz) (417 MiB) (sha256 660ceaec19740b0fda8dea028bb609edb9a4fd4cd3fc435b383fa8e4e7926db5)

Please consider [sponsoring Zig](https://github.com/users/andrewrk/sponsorship) to help cover the cost of hosting this large file. See also [[How to build LLVM, libclang, and liblld from source]].

Unzip this file to a directory with no spaces, such as `C:\Users\Andy\`. It contains a single directory, so when you do this the full path will be e.g. `C:\Users\Andy\llvm+clang-9.0.0-win64-msvc-mt\`.

Install [Build Tools for Visual Studio 2019](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2019). Be sure to select "C++ build tools" when prompted.
 * You must additionally check the optional component labeled **C++ ATL for v142 build tools**.

Install [CMake](http://cmake.org).

Use [git](https://git-scm.com/) to clone the zig repository to a path with no spaces, e.g. `C:\Users\Andy\zig`.

Using the start menu, run **x64 Native Tools Command Prompt for VS 2019** and execute these commands, replacing `C:\Users\Andy` with the correct value.

```
mkdir C:\Users\Andy\zig\build-release
cd C:\Users\Andy\zig\build-release
"c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G "Visual Studio 16 2019" -A x64 -DCMAKE_PREFIX_PATH=C:\Users\Andy\llvm+clang-9.0.0-win64-msvc-mt -DCMAKE_BUILD_TYPE=Release
msbuild -p:Configuration=Release INSTALL.vcxproj
```

You now have the `zig.exe` binary at `bin\zig.exe` and you can run the tests:

```
bin\zig.exe build test
```

This can take a long time. For tips & tricks on using the test suite, see [Contributing](https://github.com/ziglang/zig/blob/master/CONTRIBUTING.md#editing-source-code).

Note: In case you get the error "llvm-config not found" (or similar), make sure that you have **no** trailing slash (`/` or `\`) at the end of the `-DCMAKE_PREFIX_PATH` value. 
