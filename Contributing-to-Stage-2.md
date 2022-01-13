# Building Stage 2
Note that zig built with some standard LLVM distributions (e.g. LLVM from the official apt repository) can't build stage2 because some files are missing.
A good option to have a working LLVM and to cut build times is to use [zig-bootstrap](https://github.com/ziglang/zig-bootstrap) but commenting all the lines after [line 47](https://github.com/ziglang/zig-bootstrap/blob/4ed79aefb7a58a6d642f47a81e1ef04fd164042b/build#L47) (as of current zig-bootstrap master).
This will build just LLVM, clang and lld with the correct options inside `zig-bootstrap/out/host`. Then compile zig stage 1 from master as detailed inside the [relevant wiki paragraph](https://github.com/ziglang/zig/wiki/Building-Zig-From-Source#option-a-use-your-system-installed-build-tools), using the full path to `zig-bootstrap/out/host` as `CMAKE_PREFIX_PATH`.
Stage 2 can then be built with
```
zig build --zig-lib-dir lib --prefix $(pwd)/stage2 -Denable-llvm -Dconfig_h=build/config.h
```
The following sections will assume that stage 2 has been built inside the stage2 subdirectory.

# Table of WIP behavior tests

The current effort is aimed at making the stage 2 compiler pass the behavior tests (see [`test/behavior.zig`](https://github.com/ziglang/zig/blob/master/test/behavior.zig) and [`test/behavior/*.zig`](https://github.com/ziglang/zig/tree/master/test/behavior)).

Tests are currently organized in this manner:
- tests that pass for stage 1, stage 2 LLVM and stage 2 C Backend (CBE)
- tests that pass for stage 1 and stage 2 LLVM
- tests that pass for stage 1 only

## Directions specific to the C backend

 - The C backend is being worked on by multiple people, **please see [what tests are taken](https://github.com/ziglang/zig/wiki/C-Backend-Behavioral-Tests-signup-sheet) before working on one, and mark yourself when tackling a new one**. Unfortunately, a new passing test could make unrelated tests pass, which means that some duplicated work could happen anyway.
  - In order to allow iterative progress, it's acceptable to resort to compiler intrinsics and [language extensions](https://clang.llvm.org/docs/LanguageExtensions.html), but this should be loudly signaled with e.g. a `TODO` or `FIXME` comment and, for bonus points, tracked in an issue report. The goal is to produce portable C, so keep this to a minimum. For outstanding examples of this, check how the compilation output preamble from [`zig.h`](https://github.com/ziglang/zig/blob/master/src/link/C/zig.h) implements some internals such as `zig_breakpoint()`.
  - If you're looking for a practical example of hacking on the C backend, see this recording of @andrewrk's livestream: https://vimeo.com/640198169

# Using stage 2 compiler
## Using the LLVM backend
### Building a single source file
```
./stage2/bin/zig build-exe -fLLVM source.zig
```
### Testing a single source file
```
./stage2/bin/zig test -fLLVM source.zig
```
### Emitting binary for a source file test section
```
./stage2/bin/zig test --test-no-exec -femit-bin=test-source -fLLVM source.zig # Run with ./test-source
```

## Using the C backend
### Building a single source file
The following will output the `source.c` which is `source.zig` compiled with the C backend:
```
./stage2/bin/zig build-exe -ofmt=c source.zig
```
### Testing a single source file
```
./stage2/bin/zig test -ofmt=c source.zig
```
### Emitting C source for a source file test section
```
./stage2/bin/zig test --test-no-exec -femit-bin=source.c -ofmt=c source.zig
```

**Note**: For tests that use `@cImport` additional flags are needed, e.g.:
```
./stage2/bin/zig test -ofmt=c -I test/ -lc source.zig
```

## Using the native backend
Refer to the commands in the LLVM backend section, but always omit the -fLLVM flag, e.g.
```
./stage2/bin/zig build-exe source.zig
```

Currently only x86_64, arm, aarch64, and riscv native backends are being developed. To target a different backend, use the `-target` flag, e.g.:
```
./stage2/bin/zig build-exe -target aarch64-linux-gnu source.zig
```
When the -target option is not specified, the same backend as the host (usually x86-64) will be used. Available targets can be listed using `./build/stage2 zig targets`.

The LLVM backend can also target a different target other than the host system, e.g.:
```
./stage2/bin/zig build-exe -fLLVM -target aarch64-linux-gnu source.zig
```