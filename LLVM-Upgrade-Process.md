This page documents how to update the Zig source code to a new LLVM version.

master branch is always the latest LLVM release. The `llvmX` branch is kept up to date with the next LLVM release. The day that LLVM is released, we merge the `llvmX` branch into master, and start the next `llvmX` branch, following this upgrade process. 

## Before the LLVM Release

 * Start the `llvmX` branch.
 * Update the version numbers in:
    - `README.md`
    - `cmake/Findllvm.cmake`
    - `cmake/Findclang.cmake`
    - `cmake/Findlld.cmake`
 * Do a git diff in [LLVM's compiler-rt project](https://github.com/llvm/llvm-project) to find out what has changed since the previous release. Apply the relevant changes to Zig.
 * Build LLVM, Clang, and LLD in Debug mode. This can be prohibitively slow; it's OK to do Debug LLVM + Release Clang/LLD instead. But the point here is we need LLVM assertions on.
 * Update `src/zig_clang_*.cpp` to the new versions. Before updating, replace each file with the original file from the old LLVM version and do a `git diff` to see what patches have been applied by Zig. Then update each file to the versions from the new LLVM, and re-apply the patches as necessary.
    * `src/zig_clang_driver.cpp` corresponds to `llvm-project/clang/tools/driver/driver.cpp`
    * `src/zig_clang_cc1_main.cpp` corresponds to `llvm-project/clang/tools/driver/cc1_main.cpp`
    * `src/zig_clang_cc1as_main.cpp` corresponds to `llvm-project/clang/tools/driver/cc1as_main.cpp`
 * Update `lib/include/` to the latest `clang_install_prefix/lib/clang/X.Y.Z/include/`.
 * Update `lib/libcxx/`, `lib/libcxxabi`, and `lib/libunwind` to the latest `llvm-project/` respective directory. Only the `include/` directory, `src/` directory, and `LICENSE.txt` are copied. CMake files are not copied. Update `src/libunwind.zig` and `src/libcxx.zig` to have a correct list of files.
 * Run [llvm-target-details-generator](https://github.com/ziglang/zig-llvm-target-details-generator) with the new LLVM version. Commit the diff to the repository. Referencing the generated diff, update CPUs and target features in the Zig source repo correspondingly. Think carefully before deleting things, since this could represent breaking changes.
 * Update the CI scripts to the new version.
 * Update the static asserts at the bottom of `src/zig_llvm.cpp`
 * Run `tools/update_clang_options.zig` and use it to update the file `src-self-hosted/clang_options_data.zig`.
 * Build zig in debug mode with the debug mode llvm, clang, lld, and run the full Zig test suite. This takes several hours, but ensures that zig is not tripping any LLVM asserts.

## After the LLVM Release

 * Merge the `llvmX` branch into `master`.
 * Update [[How to build LLVM, libclang, and liblld from source]].
 * Update [[Building Zig on Windows]].
 * Update [[development with nix]].

