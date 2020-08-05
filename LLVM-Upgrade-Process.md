This page documents how to update the Zig source code to a new LLVM version.

 1. Update the version numbers in:
    1. `README.md`
    2. `cmake/Findllvm.cmake`
    3. `cmake/Findclang.cmake`
    4. `cmake/Findlld.cmake`
 2. Do a git diff in [LLVM's compiler-rt project](https://github.com/llvm/llvm-project) to find out what has changed since the previous release. Apply the relevant changes to Zig.
 3. Build LLVM, clang, and LLD in debug mode.
 4. Update `src/zig_clang_*.cpp` to the new versions. Before updating, replace each file with the original file from the old LLVM version and do a `git diff` to see what patches have been applied by Zig. Then update each file to the versions from the new LLVM, and re-apply the patches as necessary.
    * `src/zig_clang_driver.cpp` corresponds to `llvm-project/clang/tools/driver/driver.cpp`
    * `src/zig_clang_cc1_main.cpp` corresponds to `llvm-project/clang/tools/driver/cc1_main.cpp`
    * `src/zig_clang_cc1as_main.cpp` corresponds to `llvm-project/clang/tools/driver/cc1as_main.cpp`
 5. Update `lib/include/` to the latest `clang_install_prefix/lib/clang/X.Y.Z/include/`.
 6. Update `lib/libcxx/`, `lib/libcxxabi`, and `lib/libunwind` to the latest `llvm-project/` respective directory. Only the `include/` directory, `src/` directory, and `LICENSE.txt` are copied. Cmake files are not copied. Update `src/install_files.h` to have a correct list of files.
 7. Run [llvm-target-details-generator](https://github.com/ziglang/zig-llvm-target-details-generator) with the new LLVM version. Commit the diff to the repository. Using the generated diff, update CPUs and target features in the Zig source repo correspondingly. Think carefully before deleting things, since this could represent breaking changes.
 8. Update the CI scripts to the new version
 9. Update [[How to build LLVM, libclang, and liblld from source]]
 10. Update [[Building Zig on Windows]]
 11. Update the static asserts at the bottom of src/zig_llvm.cpp
 12. Run `tools/update_clang_options.zig` and use it to update the file `src-self-hosted/clang_options_data.zig`.
 13. Build zig in debug mode with the debug mode llvm, clang, lld, and run the full Zig test suite. This takes several hours, but ensures that zig is not tripping any LLVM asserts.

master branch is always the latest LLVM release. The `llvmX` branch is kept up to date with the next LLVM release. The day that LLVM is released, we merge the `llvmX` branch into master, and start the next `llvmX` branch, following this upgrade process. 