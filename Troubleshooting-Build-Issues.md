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

## Arch Linux, Gentoo, Fedora 32+

The Clang packages in these distributions do not contain static libraries, which Zig tries to use by default.
Instead, one should link against the shared lib `libclang-cpp.so`.

For building stage1, set `ZIG_PREFER_CLANG_CPP_DYLIB`:

```
cmake .. -DZIG_PREFER_CLANG_CPP_DYLIB=true
```
For building stage2, pass `-Dstatic-llvm=false` to Zig.

## Gentoo

If you get the message `: CommandLine Error: Option 'mc-relax-all' registered more than once!`, you're affected by the issue discussed in https://reviews.llvm.org/D75579. The fix has not (yet?) been included in clang 10.0.1. As a workaround, you can build clang with the line `add_clang_subdirectory(handle-llvm)` removed from clang/tools/clang-fuzzer/CMakeLists.txt.

Additionally, you'll need lld's .a and .h files, which aren't installed by Gentoo's lld ebuild, so you'll need to modify your lld ebuild file to not delete them.

## Ubuntu/Debian

The LLVM repositories/packages are listed at https://apt.llvm.org/. The following packages that are not listed must be installed: `liblld-12-dev`, `libclang-12-dev` (as well as at least `libllvm12` from the listed packages).

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

## Still not working?

If you are failing at the last stage ```[99%] Building self-hosted component /PATHTOZIG/zig/build/zig1.o```, make sure you have enough memory, as the process can take more 10Gib of memory.

Log on to one of the [[Community]] spaces and ask for help.