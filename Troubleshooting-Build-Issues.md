### Troubleshooting

If you get one of these:

```
undefined reference to `_ZNK4llvm17SubtargetFeatures9getStringB5cxx11Ev'
undefined reference to `llvm::SubtargetFeatures::getString() const'
```

This is because of
[C++'s Dual ABI](https://gcc.gnu.org/onlinedocs/libstdc++/manual/using_dual_abi.html).
Most likely LLVM was compiled with one compiler while Zig was compiled with a
different one, for example GCC vs clang.

To fix this, you have 2 options:

 * Compile Zig with the same compiler that LLVM was compiled with.
 * Add `-DZIG_LLVM_OLD_CXX_ABI=yes` to the cmake configure line.

