**Note**: If you want the latest build of Zig on Windows, you can get it from [the download page](https://ziglang.org/download/).

***

Here is llvm and clang x64 built with MSVC in release mode:

* [llvm+clang-7.0.0-win64-msvc-release.tar.xz](https://ziglang.org/deps/llvm%2bclang-7.0.0-win64-msvc-release.tar.xz) (290 MB) (sha256 b2d7aa01969b379c0e5de6776d75112010170a01bc21aa1e2208ff00e0ad97f3)

Please consider [donating $1/month](https://www.patreon.com/andrewrk) to help cover the cost of hosting this large file. Or you can [build LLVM and libclang from source](https://github.com/ziglang/zig/wiki/How-to-build-LLVM,-libclang,-and-liblld-from-source).

Unzip this file to a directory with no spaces, such as `C:\Users\Andy\`. It contains a single directory, so when you do this the full path will be e.g. `C:\Users\Andy\llvm+clang-7.0.0-win64-msvc\`.

Install [Visual Studio Community 2017 (version 15.8)](https://my.visualstudio.com/Downloads?q=visual%20studio%202017%2015.8&wt.mc_id=o~msft~vscom~older-downloads).

Install [CMake](http://cmake.org).

Use [git](https://git-scm.com/) to clone the zig repository to a path with no spaces, e.g. `C:\Users\Andy\zig`.

Using the start menu, run **x64 Native Tools Command Prompt for VS 2017** and execute these commands, replacing `C:\Users\Andy` with the correct value.

```
mkdir C:\Users\Andy\zig\build-release
cd C:\Users\Andy\zig\build-release
"c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G"Visual Studio 15 2017 Win64" -DCMAKE_INSTALL_PREFIX=C:\Users\Andy\zig\build-release -DCMAKE_PREFIX_PATH=C:\Users\Andy\llvm+clang-7.0.0-win64-msvc-release -DCMAKE_BUILD_TYPE=Release
msbuild -p:Configuration=Release INSTALL.vcxproj
```

You now have the `zig.exe` binary at `bin\zig.exe` and you should run the tests to make sure they pass:

```
bin\zig.exe build --build-file ..\build.zig test
```