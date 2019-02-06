If you want to run the stage1 compiler on the target (rather than only cross compiling for the target), add the `os.cpp` code for the new target. You can probably let compile errors guide your efforts here.

Add the startup code in `std/special/bootstrap.zig` for the new target.
This code is responsible for the real executable entry point, calling `main()` and making the `exit` syscall when main returns.

Update the C integer types to be the correct size for the target. This one will be obvious, causing a panic when you try to build if you don't do it. The function is `target_c_type_size_in_bits`.

You may need to audit the C ABI compatibility for the new target. Fortunately we have some C ABI tests now, which you can run with `bin/zig build --build-file ../test/stage1/c_abi/build.zig test`. Unfortunately they are disabled on Windows, MacOS, and non x86_64. These are all bugs (or lack of features depending on how you look at it) and we would like to improve the coverage of the C ABI tests. These tests also require ability to run stage1 compiler on the target.

Make sure that `c_longdouble` codegens the correct floating point value. This should ideally be handled in the C ABI tests mentioned above. Once that is true we can remove this step from the wiki.

Lots of places in the standard library will call `@compileError("Unsupported OS")`. You'll have to replace this with your target-specific implementation. You should be able to try running the behavior tests, and then the standard library tests, which will reveal these compile errors.