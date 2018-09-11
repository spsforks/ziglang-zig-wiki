This page documents how to update the Zig source code to a new LLVM version.

 1. Update the version numbers in:
    1. README.md
    2. cmake/Findllvm.cmake
    3. cmake/Findclang.cmake
    4. cmake/Findlld.cmake
 2. Do an svn diff in LLVM's compiler-rt project to find out what has changed since the previous release. Apply the relevant changes to Zig.
 3. Upgrade `deps/lld` and `deps/lld-prebuilt` to the new LLD code, and then reapply any outstanding patches that we have against upstream. You can find out about outstanding patches with git, since they are done in separate commits.
 4. Update `c_headers/` to the latest `clang_release_XY/build/lib/clang/X.Y.Z/include/`
 5. Update the CI scripts to the new version
 6. Update [[How to build LLVM, libclang, and liblld from source]]
 7. Update [[Building Zig on Windows]]
 8. Build LLVM, clang, and LLD in debug mode, applying Zig's outstanding patches to LLD. Build zig in debug mode against these libraries and run the full Zig test suite. This takes several hours.