Master branch build status: [![Build Status](https://dev.azure.com/ziglang/zig/_apis/build/status/ziglang.zig?branchName=master)](https://dev.azure.com/ziglang/zig/_build/latest?definitionId=1&branchName=master)

## [Troubleshooting Build Issues](https://github.com/ziglang/zig/wiki/Troubleshooting-Build-Issues)

## [[Building Zig on Windows]]

# Stage 1: Build Zig from C++ Source Code

This step must be repeated when you make changes to any of the C++ source code.

## Option A: Use Your System Installed Build Tools

### Dependencies:

 * cmake >= 2.8.12
 * gcc >= 7.0.0 or clang >= 6.0.0
 * LLVM, Clang, LLD development libraries == 12.x, compiled with the same gcc or clang version above
   - Use the system package manager, or [build from source](https://github.com/ziglang/zig/wiki/How-to-build-LLVM,-libclang,-and-liblld-from-source#posix).

### Instructions

```sh
mkdir build
cd build
cmake ..
make install
```

Please be aware of the handy cmake variable `CMAKE_PREFIX_PATH`. For example, macOS users may want to use `cmake .. -DCMAKE_PREFIX_PATH=$(brew --prefix llvm)`.

Note: To compile in release mode use the `-DCMAKE_BUILD_TYPE=Release` flag.

## Option B: Use a Pre-Built Zig Binary

### Dependencies

 * A previous build of Zig, `0.7.0+a01d55e80` or newer.
 * LLVM, Clang, and LLD libraries built using Zig. The easiest way to obtain this is to use [zig-bootstrap](https://github.com/ziglang/zig-bootstrap).

### Instructions

```sh
zig build -Dstage1 --search-prefix $SEARCH_PREFIX
cmake ..
make install
```

Where `$SEARCH_PREFIX` is the path that contains, for example, `include/llvm/Pass.h` and `lib/libLLVMCore.a`.

Remember! For Option B, these libraries *must be produced by `zig cc` / `zig c++`* - **not** by your system C/C++ compiler. If you are annoyed by this, welcome to the club, please enjoy this extra reason to hate C++ on the house.

# Stage 2: Build Self-Hosted Zig from Zig Source Code

If you intend to develop the stage2 compiler itself, then continue onward. Otherwise, use the stage1 compiler built in the previous step for general Zig usage (stage2 is not ready yet to be used other than experimental usage.)

Now we use the stage1 binary:

```
zig build --prefix $(pwd)/stage2 -Denable-llvm
```

This produces `stage2/bin/zig` which can be used for testing and development.
Once it is feature complete, it will be used to build stage 3 - the final compiler
binary.

This is the main effort of the 0.8.0 release cycle - the stage2 compiler. There are quite a few build options which can aid your development experience. Have a look with `zig build --help`.

# Stage 3: Rebuild Self-Hosted Zig Using the Self-Hosted Compiler

*Note: Stage 2 compiler is not yet able to build Stage 3. Building Stage 3 is
not yet supported.*

Once the self-hosted compiler can build itself, this will be the actual
compiler binary that we will install to the system. Until then, users should
use stage 1.

## Debug / Development Build

```
stage2/bin/zig build
```

This produces `zig-cache/bin/zig`.

## Release / Install Build

```
stage2/bin/zig build install -Drelease
```

