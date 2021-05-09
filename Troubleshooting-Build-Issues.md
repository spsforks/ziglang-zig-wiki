### Troubleshooting

#### Dual-Abi Linking

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

#### Arch Linux

The Clang package distributed via official Pacman sources isn't built with static libraries,
therefore, it is not possible to statically link against individual Clang libs. Instead, one
should link against the shared lib `libclang-cpp.so`. Thus, when building on Archlinux, you
need to pass `ZIG_PREFER_CLANG_CPP_DYLIB` flag set to true like so:

```
cmake .. -DZIG_PREFER_CLANG_CPP_DYLIB=true
```

#### Gentoo

Like Arch Linux' clang package, Gentoo's Clang is built without static libraries, so you'll need to pass `-DZIG_PREFER_CLANG_CPP_DYLIB=true` to cmake.

If you get the message `: CommandLine Error: Option 'mc-relax-all' registered more than once!`, you're affected by the issue discussed in https://reviews.llvm.org/D75579. The fix has not (yet?) been included in clang 10.0.1. As a workaround, you can build clang with the line `add_clang_subdirectory(handle-llvm)` removed from clang/tools/clang-fuzzer/CMakeLists.txt.

Additionally, you'll need lld's .a and .h files, which aren't installed by Gentoo's lld ebuild, so you'll need to modify your lld ebuild file to not delete them.

#### Fedora 32+

Starting with version 32, Fedora will no longer ship individual component libraries in the `clang-libs` package instead requiring users to link against  `libclang-cpp.so`. Thus when building on Fedora version 32 or greater you'll need to pass `-DZIG_PREFER_CLANG_CPP_DYLIB=true` to cmake.

#### Ubuntu/Debian

The LLVM repositories/packages are listed at https://apt.llvm.org/. The following packages that are not listed must be installed: `liblld-12-dev`, `libclang-12-dev` (as well as at least `libllvm12` from the listed packages).

### Still not working?

Log on to one of the [[Community]] spaces and ask for help.