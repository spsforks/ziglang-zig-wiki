## Building Stage 2
See [relevant wiki paragraph](https://github.com/ziglang/zig/wiki/Building-Zig-From-Source#stage-2-build-self-hosted-zig-from-zig-source-code) to build stage 2 compiler. The following sections will assume that stage 2 has been built inside the stage2 subdirectory.

## Using stage 2 compiler
### Building a single source file (LLVM backend)
```
./stage2/bin/zig build-exe -fLLVM source.zig
```
### Building a single source file (C backend)
```
./stage2/bin/zig build-exe -ofmt=c source.zig
```