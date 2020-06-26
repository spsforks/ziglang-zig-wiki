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
