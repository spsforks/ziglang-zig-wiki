This page documents how to update the Zig source code to a new LLVM version.

 1. Update the version numbers in:
    1. `README.md`
    2. `cmake/Findllvm.cmake`
    3. `cmake/Findclang.cmake`
    4. `cmake/Findlld.cmake`
 2. Do an svn diff in LLVM's compiler-rt project to find out what has changed since the previous release. Apply the relevant changes to Zig.
 3. Build LLVM, clang, and LLD in debug mode, applying Zig's outstanding patches to LLD.
 4. Upgrade `deps/lld` and `deps/lld-prebuilt` to the new LLD code, and then reapply any outstanding patches that we have against upstream. You can find out about outstanding patches with git, since they are done in separate commits.
 5. Update `c_headers/` to the latest `clang_release_XY/build-debug/lib/clang/X.Y.Z/include/`. Then update `CMakeLists.txt` to have the new list of files.
 6. Update the CI scripts to the new version
 7. Update [[How to build LLVM, libclang, and liblld from source]]
 8. Update [[Building Zig on Windows]]
 9. Build zig in debug mode with the debug mode llvm, clang, patched lld, and run the full Zig test suite. This takes several hours.

master branch is always the latest LLVM release. The `llvmX` branch is kept up to date with the next LLVM release. The day that LLVM is released, we merge the `llvmX` branch into master, and start the next `llvmX` branch, following this upgrade process. 