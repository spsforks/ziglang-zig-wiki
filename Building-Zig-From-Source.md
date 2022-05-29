Master branch build status: [![Build Status](https://dev.azure.com/ziglang/zig/_apis/build/status/ziglang.zig?branchName=master)](https://dev.azure.com/ziglang/zig/_build/latest?definitionId=1&branchName=master)

[Troubleshooting Build Issues](https://github.com/ziglang/zig/wiki/Troubleshooting-Build-Issues)

## Microsoft Windows

[[Building Zig on Windows]]

# Stage 1: Build Zig from C++ Source Code

This step must be repeated when you make changes to any of the C++ source code.

## Option A: Use Your System Installed Build Tools

### Dependencies:

 * cmake >= 2.8.12
 * gcc >= 7.0.0 or clang >= 6.0.0
 * LLVM, Clang, LLD development libraries == 13.x, compiled with the same gcc or clang version above
   - Use the system package manager, or [build from source](https://github.com/ziglang/zig/wiki/How-to-build-LLVM,-libclang,-and-liblld-from-source#posix).

### Instructions

```sh
mkdir build
cd build
cmake ..
make install
```

Please be aware of the handy cmake variable `CMAKE_PREFIX_PATH`. For example, macOS users may want to use `cmake .. -DCMAKE_PREFIX_PATH=$(brew --prefix llvm)`.

Note: On macOS, since LLVM 12.0 release, Homebrew's packaged LLVM reports itself as a dynamic dependency while Zig's config system will expect a static dependency. This can lead to unexpected errors when trying to compile C/C++ with Zig. To force static linking of LLVM, use `-DZIG_STATIC_LLVM=on` flag.

Note: See this page for
[Troubleshooting Build Issues](https://github.com/ziglang/zig/wiki/Troubleshooting-Build-Issues)

Note: To compile in release mode use the `-DCMAKE_BUILD_TYPE=Release` flag.

## Option B: Use a Pre-Built Zig Binary

### Dependencies

 * A previous build of Zig, `0.10.0-dev.2025+f65ca80bb` or newer. If the language or std lib changed too much since this version, then this strategy will fail with compilation errors.
 * LLVM, Clang, and LLD libraries built using Zig. The easiest way to obtain this is to use [zig-bootstrap](https://github.com/ziglang/zig-bootstrap), which creates the directory `out/host`, to be used as `$OLD_ZIG_PREFIX` in the following command:

### Instructions

```sh
"$OLD_ZIG_PREFIX/bin/zig" build -p stage1 -Dstage1 -Domit-stage2 --search-prefix "$OLD_ZIG_PREFIX" --zig-lib-dir "$OLD_ZIG_PREFIX/lib"
```

Where `$SEARCH_PREFIX` is the path that contains, for example, `include/llvm/Pass.h` and `lib/libLLVMCore.a`.

Remember! For Option B, these libraries *must be produced by `zig cc` / `zig c++`* - **not** by your system C/C++ compiler. If you are annoyed by this, welcome to the club.

In the following steps, carry the `--search-prefix "$OLD_ZIG_PREFIX"` parameters to each `zig build` command but not the others.

# Stage 2: Build Self-Hosted Zig from Zig Source Code

If you intend to develop the stage2 compiler itself, then continue onward. Otherwise, use the stage1 compiler built in the previous step for general Zig usage (stage2 is not ready yet to be used other than experimental usage; see [#89](https://github.com/ziglang/zig/issues/89).)

Now we use the stage1 binary produced from the previous step:

```
./stage1/bin/zig build -p stage2 -Denable-llvm
```

This produces `stage2/bin/zig` which can be used for testing and development.

There are quite a few build options which can aid your development experience. Have a look with `zig build --help`.

# Stage 3: Rebuild Self-Hosted Zig Using the Self-Hosted Compiler

The final step is to make the self-hosted compiler rebuild itself. This produces the actual compiler binary that we will install to the system.

## Debug / Development Build

```
stage2/bin/zig build -p stage3 -Denable-llvm
```

This produces `stage3/bin/zig`.

## Release / Install Build

```
stage2/bin/zig build -Drelease -Denable-llvm
```