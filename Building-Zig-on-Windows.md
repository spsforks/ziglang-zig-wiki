**Note**: If you want the latest build of Zig on Windows, you can get it from [the download page](https://ziglang.org/download/).

***

Here is llvm and clang x64 built with MSVC in release mode:

* [llvm+clang-8.0.0-win64-msvc-release.tar.xz](https://ziglang.org/deps/llvm%2bclang-8.0.0-win64-msvc-release.tar.xz) (307 MiB) (sha256 cab981f1c170cc818bc814d980b81faf91bbd5d5ad7059aeee01abe1b92e0084)

Please consider [supporting Zig on Patreon](https://www.patreon.com/andrewrk) to help cover the cost of hosting this large file. Or you can [build LLVM and libclang from source](https://github.com/ziglang/zig/wiki/How-to-build-LLVM,-libclang,-and-liblld-from-source).

Unzip this file to a directory with no spaces, such as `C:\Users\Andy\`. It contains a single directory, so when you do this the full path will be e.g. `C:\Users\Andy\llvm+clang-8.0.0-win64-msvc\`.

Install [Build Tools for Visual Studio 2019](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2019).

Install [CMake](http://cmake.org).

Use [git](https://git-scm.com/) to clone the zig repository to a path with no spaces, e.g. `C:\Users\Andy\zig`.

Using the start menu, run **x64 Native Tools Command Prompt for VS 2019** and execute these commands, replacing `C:\Users\Andy` with the correct value.

```
mkdir C:\Users\Andy\zig\build-release
cd C:\Users\Andy\zig\build-release
"c:\Program Files\CMake\bin\cmake.exe" .. -G "Visual Studio 16 2019" -A x64 -DCMAKE_INSTALL_PREFIX=C:\Users\Andy\zig\build-release -DCMAKE_PREFIX_PATH=C:\Users\Andy\llvm+clang-8.0.0-win64-msvc-release -DCMAKE_BUILD_TYPE=Release
msbuild -p:Configuration=Release INSTALL.vcxproj
```

You now have the `zig.exe` binary at `bin\zig.exe` and you should run the tests to make sure they pass:

```
bin\zig.exe build --build-file ..\build.zig test
```