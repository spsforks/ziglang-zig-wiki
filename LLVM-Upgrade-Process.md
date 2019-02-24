This page documents how to update the Zig source code to a new LLVM version.

 1. Update the version numbers in:
    1. `README.md`
    2. `cmake/Findllvm.cmake`
    3. `cmake/Findclang.cmake`
    4. `cmake/Findlld.cmake`
 2. Do an git diff in [LLVM's compiler-rt project](https://github.com/llvm/llvm-project) to find out what has changed since the previous release. Apply the relevant changes to Zig.
 3. Build LLVM, clang, and LLD in debug mode, applying Zig's outstanding patches to LLD.
 4. Upgrade `deps/lld` and `deps/lld-prebuilt` to the new LLD code, and then reapply any outstanding patches that we have against upstream. You can find out about outstanding patches with git, since they are done in separate commits.
 5. Update `src/zig_clang_*.cpp` to the new versions. Before updating, replace each file with the original file from the old LLVM version and do a `git diff` to see what patches have been applied by Zig. Then update each file to the versions from the new LLVM, and re-apply the patches as necessary.
    * `src/zig_clang_driver.cpp` corresponds to `llvm-project/clang/tools/driver/driver.cpp`
    * `src/zig_clang_cc1_main.cpp` corresponds to `llvm-project/clang/tools/driver/cc1_main.cpp`
    * `src/zig_clang_cc1as_main.cpp` corresponds to `llvm-project/clang/tools/driver/cc1as_main.cpp`
 6. Update `c_headers/` to the latest `clang_release_XY/build-debug/lib/clang/X.Y.Z/include/`. Then update `CMakeLists.txt` to have the new list of files.
 7. Update the CI scripts to the new version
 8. Update [[How to build LLVM, libclang, and liblld from source]]
 9. Update [[Building Zig on Windows]]
 10. Update the static asserts at the bottom of src/zig_llvm.cpp
 11. Build zig in debug mode with the debug mode llvm, clang, patched lld, and run the full Zig test suite. This takes several hours.

master branch is always the latest LLVM release. The `llvmX` branch is kept up to date with the next LLVM release. The day that LLVM is released, we merge the `llvmX` branch into master, and start the next `llvmX` branch, following this upgrade process. 