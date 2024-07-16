Imagine this: You're on a new M1 MacBook<sup>1</sup>. You need to compile a CMake project to run on your friend's Windows x86-64 PC. How do you do that? Use `zig cc` and friends! 🚀

Here's an example project that only works on Windows:

<div><code>main.c</code></div>

```c
#include <windows.h>

int APIENTRY WinMain(HINSTANCE hInst, HINSTANCE hInstPrev, PSTR cmdline, int cmdshow)
{
    return MessageBox(NULL, "hello, world", "caption", 0);
}
```

<div><code>CMakeLists.txt</code></div>

```cmake
cmake_minimum_required(VERSION 3.29)
project(hello-world LANGUAGES C)
add_executable(hello-world WIN32 main.c)
```

So how can we compile it for Windows x86-64 from our M1 MacBook<sup>1</sup>? You'll need to add some supporting infrastructure first. `CMAKE_AR="zig;ar"` and `CMAKE_RANLIB="zig;ranlib"` don't work. `CMAKE_AR` and `CMAKE_RANLIB` don't support commands with arguments so we need to use a wrapper `./zig-ar` script instead. 🤷‍♀️

<div><code>zig-ar</code> (<code>chmod +x</code>)</div>

```sh
exec zig ar "$@"
```

<div><code>zig-ar.cmd</code></div>

```cmd
@zig ar %*
```

<div><code>zig-ranlib</code> (<code>chmod +x</code>)</div>

```sh
exec zig ranlib "$@"
```

<div><code>zig-ranlib.cmd</code></div>

```cmd
@zig ranlib %*
```

You now have everything ready to run the CMake configure & build steps! 🎉 Don't forget to pass all the appropriate flags to compile for Windows x86-64 using the Zig toolchain! 😉

<table><td>

```sh
cmake \
  -DCMAKE_SYSTEM_NAME="Windows" \
  -DCMAKE_SYSTEM_PROCESSOR="x86_64" \
  -DCMAKE_ASM_COMPILER="zig;cc" \
  -DCMAKE_ASM_COMPILER_TARGET="x86_64-windows-gnu" \
  -DCMAKE_C_COMPILER="zig;cc" \
  -DCMAKE_C_COMPILER_TARGET="x86_64-windows-gnu" \
  -DCMAKE_CXX_COMPILER="zig;c++" \
  -DCMAKE_CXX_COMPILER_TARGET="x86_64-windows-gnu" \
  -DCMAKE_AR="./zig-ar" \
  -DCMAKE_RANLIB="./zig-ranlib" \
  -B build
```

<tr><td>

```
-- The C compiler identification is Clang 18.1.6
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - failed
-- Check for working C compiler: /somewhere/zig
-- Check for working C compiler: /somewhere/zig - works
-- Detecting C compile features
-- Detecting C compile features - done
-- Configuring done (5.5s)
-- Generating done (0.0s)
-- Build files have been written to: /somewhere/build
```

</table>

💡 You can change the `CMAKE_C_COMPILER_TARGET` to any supported Zig target. Use `zig targets` to see them all.

**Do I _have to_ specify the `CMAKE_SYSTEM_NAME` and `CMAKE_SYSTEM_PROCESSOR`?** Yeah. It triggers a cascade of configuration that configures the `.exe` output suffix, sets `WIN32=1`, and a lot more.

**What about platform-specific file extensions for `./zig-ar`?** Windows will helpfully scan all extensions from `%PATHEXT%` (`.COM;.EXE;.BAT;.CMD;...`) for you. That's why you can run `zig` in your PowerShell/CMD prompt when the actual file is called `zig.exe`.

**What's the `zig;cc` syntax?** `CMAKE_C_COMPILER` accepts a list. Lists in CMake are just strings with semicolon delimiters.

**How does `CMAKE_C_COMPILER_TARGET` make its way into `zig cc --target`?** `zig cc` is detected as Clang which is known by CMake to support a `--target` option.

Now that the project is configured with all the compiler settings and other magic✨ we can run the build step:

<table><td>

```sh
cmake --build build
```

<tr><td>

```
[ 50%] Building C object CMakeFiles/hello-world.dir/main.c.obj
[100%] Linking C executable hello-world.exe
[100%] Built target hello-world
```

</table>

Tada! 🥳 You have now built a Windows x86-64 `.exe` file from your M1 MacBook<sup>1</sup>!

<table><td>

```sh
./build/hello-world.exe
```

<tr><td>

![hello world alert box](https://github.com/user-attachments/assets/61a10f60-8eab-4ad7-97fc-9ad31a8f83b9)

</table>

You can use this on any ASM/C/C++ CMake project! 🚀 Cross-compiling has never been easier than with `zig cc` and friends. Use it in your CI pipeline to build binaries for more than just The Big 5 (Windows x86-64, macOS x86-64 & ARM64, Linux x86-64 & ARM64). You can see a bunch of Zig targets via `zig targets`.

<!-- GitHub supports footnotes natively https://github.blog/changelog/2021-09-30-footnotes-now-supported-in-markdown-fields/ but they don't seem to work in the GitHub wiki? 🤷‍♂️ -->

---

1. It doesn't have to be an M1 MacBook. The point is to emphasize that a cross-compiler is required.