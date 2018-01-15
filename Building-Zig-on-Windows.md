**Note**: If you want the latest build of Zig on Windows, you can find the [latest successful build on appveyor](https://ci.appveyor.com/project/andrewrk/zig-d3l86/history?branch=master) and then click Artifacts, and download the .zip file there.

***

Here is llvm and clang 5.0.1 x64 built with MSVC in release mode:

* [llvm+clang-5.0.1-win64-msvc-release.tar.xz](http://ziglang.org/deps/llvm%2bclang-5.0.1-win64-msvc-release.tar.xz) (248 MB) (sha256 d77ded849fe1b751cbf336087129b893b5b0d1401eeb38b0b0055911928af877)

Please [donate $1/month](https://www.patreon.com/andrewrk) to help cover the cost of hosting this large file.

Unzip this file to a directory with no spaces, such as `C:\Users\Andy\`. It contains a single directory, so when you do this the full path will be e.g. `C:\Users\Andy\llvm+clang-5.0.1-win64-msvc\`.

Install [Visual Studio Community 2015 with Update 3](https://my.visualstudio.com/Downloads?q=visual%20studio%202015&wt.mc_id=o~msft~vscom~older-downloads).

Install [CMake](http://cmake.org).

Use [git](https://git-scm.com/) to clone the zig repository to a path with no spaces, e.g. `C:\Users\Andy\zig`.

Using the start menu, run **VS2015 x64 Native Tools Command Prompt** and execute these commands, replacing `C:\Users\Andy` with the correct value.

```
> mkdir C:\Users\Andy\zig\build-release
> cd C:\Users\Andy\zig\build-release
> "c:\Program Files\CMake\bin\cmake.exe" .. -Thost=x64 -G"Visual Studio 14 2015 Win64" -DCMAKE_INSTALL_PREFIX=C:\Users\Andy\zig\build-release -DCMAKE_PREFIX_PATH=C:\Users\Andy\llvm+clang-5.0.1-win64-msvc-release -DCMAKE_BUILD_TYPE=Release
> msbuild -p:Configuration=Release INSTALL.vcxproj
```

You now have the `zig.exe` binary at `bin\zig.exe` and you should run the tests to make sure they pass:

```
bin\zig.exe build --build-file ..\build.zig test
```