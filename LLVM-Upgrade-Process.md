This page documents how to update the Zig source code to a new LLVM version.

master branch is always the latest LLVM release. The `llvmX` branch is kept up to date with the next LLVM release. The day that LLVM is released, we merge the `llvmX` branch into master, and start the next `llvmX` branch, following this upgrade process. 

## Before the LLVM Release

 * Start the `llvmX` branch.
 * Update the version numbers in:
    - `cmake/Findllvm.cmake`
    - `cmake/Findclang.cmake`
    - `cmake/Findlld.cmake`
 * Build LLVM, Clang, and LLD in Release mode with assertions on.
 * Update `src/zig_clang_*.cpp` to the new versions. Before updating, replace each file with the original file from the old LLVM version and do a `git diff` to see what patches have been applied by Zig. Then update each file to the versions from the new LLVM, and re-apply the patches as necessary.
    * `src/zig_clang_driver.cpp` corresponds to `llvm-project/clang/tools/driver/driver.cpp`
      - Be sure not to accidentally regress [#3292](https://github.com/ziglang/zig/pull/3292).
    * `src/zig_clang_cc1_main.cpp` corresponds to `llvm-project/clang/tools/driver/cc1_main.cpp`
    * `src/zig_clang_cc1as_main.cpp` corresponds to `llvm-project/clang/tools/driver/cc1as_main.cpp`
    * `src/zig_llvm-ar.cpp` corresponds to `llvm-project/llvm/tools/llvm-ar/llvm-ar.cpp`
 * Update `lib/include/` to the latest `clang_install_prefix/lib/clang/X.Y.Z/include/`.
 * Update `lib/libcxx/`, `lib/libcxxabi`, and `lib/libunwind` to the latest `llvm-project/` respective directory. Only the `include/` directory, `src/` directory, and `LICENSE.txt` are copied. CMake, README, shell scripts files are not copied. Update `src/libunwind.zig` and `src/libcxx.zig` to have a correct list of files.
 * Update `tsan` to the latest `llvm-project/compiler-rt/tsan` directory. Only the necessary files are copied. Update `src/libtsan.zig` to have a correct list of files.
 * Run `tools/update_cpu_features.zig` with the new LLVM version. Unless you have a beefy machine, you'll probably want `--single-threaded` to limit the CPU & memory resource usage of this script. After running the tool, inspect the git diff. Think carefully about deletions, since this could represent breaking changes. Modifications to the script might be needed in order to clean up the output.
 * Update the CI scripts to the new version.
 * Update the static asserts at the bottom of `src/zig_llvm.cpp`
 * Run `tools/update_clang_options.zig` and use it to update the file `src-self-hosted/clang_options_data.zig`.
 * Build zig in debug mode with asserts-enabled llvm, clang, lld, and run the full Zig test suite.

## After the LLVM Release

 * Merge the `llvmX` branch into `master`.
 * Update [[How to build LLVM, libclang, and liblld from source]].
 * Update [[Building Zig on Windows]].

