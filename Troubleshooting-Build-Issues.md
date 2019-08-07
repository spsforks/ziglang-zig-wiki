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

---
#### Unable to find clang on Ubuntu, despite having installed it

This is because you need--for some reason--to install libclang explicitly.

You can do this with the following command:
```
sudo apt-get install libclang-8-dev
```
Note that the `8` should match LLVM's version.
At the time of writing in August 2019, this is LLVM 8.