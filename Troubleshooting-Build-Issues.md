## Dual-Abi Linking

If you get one of these:

```
undefined reference to `_ZNK4llvm17SubtargetFeatures9getStringB5cxx11Ev'
undefined reference to `llvm::SubtargetFeatures::getString() const'
```

This is because of
[C++'s Dual ABI](https://gcc.gnu.org/onlinedocs/libstdc++/manual/using_dual_abi.html).
Most likely LLVM was compiled with one compiler while Zig was compiled with a
different one, for example GCC vs clang.

LLVM, Clang, and Zig must all be compiled with the same C++ compiler.

## Building stage2 and stage3

After successful building stage1, you build stage2 (or the same with stage3).
If you get one of these:

```
LLD Link... ld.lld: error: unable to find library -lclangFrontendTool
ld.lld: error: unable to find library -lclangCodeGen
...
```

Then Zig was not able to find the LLVM libs. Add the path to the config header of the stage1 build to the build command, for example:
```
./stage2/bin/zig build -p stage3 -Denable-llvm -Dconfig_h=build/config.h
```

If you get this error (confirmed on Arch Linux):

```
ld.lld: error: undefined symbol: __libc_single_threaded
```

Then it is necessary to link the `c_nonshared` library. See [#11137](/ziglang/zig/issues/11137).

## Arch Linux, Gentoo, Fedora 32+

The Clang packages in these distributions do not contain static libraries, which Zig tries to use by default.
Instead, one should link against the shared lib `libclang-cpp.so`.

For building stage1, set `ZIG_PREFER_CLANG_CPP_DYLIB`:

```
cmake .. -DZIG_PREFER_CLANG_CPP_DYLIB=true
```
For building stage2, pass `-Dstatic-llvm=false` to Zig.

Fedora also doesn't ship by default with the static libraries for lib stdc++. 
This will result in link errors for symbols like `std::string`.
You can fix this by installing the following package: `dnf install libstdc++-static`.

## Gentoo

If you get the message `: CommandLine Error: Option 'mc-relax-all' registered more than once!`, you're affected by the issue discussed in https://reviews.llvm.org/D75579. The fix has not (yet?) been included in clang 10.0.1. As a workaround, you can build clang with the line `add_clang_subdirectory(handle-llvm)` removed from clang/tools/clang-fuzzer/CMakeLists.txt.

Additionally, you'll need lld's .a and .h files, which aren't installed by Gentoo's lld ebuild, so you'll need to modify your lld ebuild file to not delete them.

## Ubuntu/Debian

The LLVM repositories/packages are listed at https://apt.llvm.org/. The following packages that are not listed must be installed: `liblld-13-dev`, `libclang-13-dev` (as well as at least `libllvm13` from the listed packages).

## macOS

Linking with Homebrew's packaged LLVM, since version 12.0, you will need to force static linking of LLVM using `-DZIG_STATIC_LLVM=on`. The full invocation will then look like:

```
cmake .. -DCMAKE_PREFIX_PATH=$(brew --prefix llvm) -DZIG_STATIC_LLVM=on
```

After upgrading LLVM, it's often necessary to remove the CMake cache and build directories:

```
rm CMakeCache.txt
rm -rf CMakeFiles
```

Then try running the cmake command again.

### M1 Macs (ARM)
On M1 Macs, static linking with homebrew LLVM does not seem to work. Omit `-DZIG_STATIC_LLVM=on` to revert to the default behavior (linking LLVM dynamically).

For similar reasons, you should also dynamically link to clang. This can be done by specifying `-DZIG_PREFER_CLANG_CPP_DYLIB=true`. Static linking is default for clang, dynamic linking is default for LLVM. 

Building with LLVM 14 has `zlib` as a new requirement. `zlib` can be installed with homebrew, but must be configured for static linking using `-DZIG_STATIC_ZLIB=ON`

Final command for dynamically linking to homebrew LLVM/Clang on M1 Macs:
```
cmake .. -DCMAKE_PREFIX_PATH=$(brew --prefix llvm) -DZIG_PREFER_CLANG_CPP_DYLIB=true -DZIG_STATIC_ZLIB=ON
```

This fixes all the strange linking issues I've had.

A prior recommendation was to use static linking for LLVM+Clang on Mac. However, the opposite seems to be true on the new M1 Macs (maybe try both if you still have issues?).

## High Memory Requirements

If you are failing at the last stage ```[99%] Building self-hosted component /PATHTOZIG/zig/build/zig1.o```, make sure you have enough memory, as the process can take more than 10GiB of memory.

Related issue: [zig0 takes too much RAM to build zig1.o](https://github.com/ziglang/zig/issues/6485)
The main development effort of Zig is focused on the self-hosted compiler, which solves this issue.
