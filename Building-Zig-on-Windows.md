Here is llvm and clang 5.0.0 x64 built with MSVC in release mode:

* [llvm+clang-5.0.0-win64-msvc-release.tar.xz](https://s3.amazonaws.com/superjoe/temp/llvm%2bclang-5.0.0-win64-msvc-release.tar.xz) (495 MB) (sha256 31e4944e66d563aaf2b45bcf9ac3590e3f7b2227b481560d216a926bbe4dd40e)

Please [donate $1/month](https://www.patreon.com/andrewrk) to help cover cost of hosting such a large file.

Unzip this file to a directory with no spaces, such as `C:\Users\Andy\`. It contains a single directory, so when you do this the full path will be e.g. `C:\Users\Andy\llvm+clang-5.0.0-win64-msvc\`.

Install [Visual Studio Community 2015 with Update 3](https://my.visualstudio.com/Downloads?q=visual%20studio%202015&wt.mc_id=o~msft~vscom~older-downloads).

Install [CMake](http://cmake.org).

Before you clone zig, make sure you have git [configured such that it will use Unix newlines "\n" instead of Windows newlines "\n\r"](https://help.github.com/articles/dealing-with-line-endings/#platform-windows), otherwise the zig compiler will complain. Run `git config --global core.autocrlf true` in the console.

Use [git](https://git-scm.com/) to clone the zig repository to a path with no spaces, e.g. `C:\Users\Andy\zig`.

Run the CMake GUI.

**Browse Source...** and select `C:\Users\Andy\zig`.

**Browse Build...** and select `C:\Users\Andy\zig\build` (this is where you cloned zig to, plus `\build`).

Click **Add Entry** and add these entries (these are examples - use the correct values):

 * `CMAKE_PREFIX_PATH` type `PATH` value `C:\Users\Andy\llvm+clang-5.0.0-win64-msvc`.
 * `CMAKE_INSTALL_PREFIX` type `PATH` value `C:\Users\Andy\zig\build`. (this is the same as the build folder you selected above)

Click **Configure** and use these options:

 * Specify the generator for this project: `Visual Studio 14 2015 Win64`
 * Optional toolset to use: `host=x64`
 * `Use the default native compilers`

Click **Generate**.

Using the start menu, run **VS2015 x64 Native Tools Command Prompt** and execute these commands:

```
cd C:\Users\Andy\zig\build
msbuild INSTALL.vcxproj
```

You now have the `zig.exe` binary at `bin\zig.exe` and you should run the tests to make sure they pass:

```
bin\zig.exe build --build-file ..\build.zig test
```