## Building Stage 2
Note that zig built with some standard LLVM distributions (e.g. LLVM from the official apt repository) can't build stage2 because some files are missing.
A good option to have a working LLVM and to cut build times is to use [zig-bootstrap](https://github.com/ziglang/zig-bootstrap) but commenting all the lines after [line 47](https://github.com/ziglang/zig-bootstrap/blob/4ed79aefb7a58a6d642f47a81e1ef04fd164042b/build#L47) (as of current zig-bootstrap master).
This will build just LLVM, clang and lld with the correct options inside `zig-bootstrap/out/host`. Then compile zig stage 1 from master as detailed inside the [relevant wiki paragraph](https://github.com/ziglang/zig/wiki/Building-Zig-From-Source#option-a-use-your-system-installed-build-tools), using the full path to `zig-bootstrap/out/host` as `CMAKE_PREFIX_PATH`.
Stage 2 can then be built with
```
zig build --zig-lib-dir lib --prefix $(pwd)/stage2 -Denable-llvm -Dconfig_h=build/config.h
```
The following sections will assume that stage 2 has been built inside the stage2 subdirectory.

## Using stage 2 compiler
### Building a single source file (LLVM backend)
```
./stage2/bin/zig build-exe -fLLVM source.zig
```
### Building a single source file (C backend)
The following will output the `source.c` which is `source.zig` compiled with the C backend:
```
./stage2/bin/zig build-exe -ofmt=c source.zig
```

### Testing a single source file (LLVM backend)
```
./stage2/bin/zig test -fLLVM source.zig
```

### Testing a single source file (C backend)
```
./stage2/bin/zig test -ofmt=c source.zig
```

### Emitting binary for a source file test section (LLVM backend)
```
./stage2/bin/zig test --test-no-exec -femit-bin=source -ofmt=c source.zig
```

### Emitting C source for a source file test section (C backend)
```
./stage2/bin/zig test --test-no-exec -femit-bin=source.c -ofmt=c source.zig
```