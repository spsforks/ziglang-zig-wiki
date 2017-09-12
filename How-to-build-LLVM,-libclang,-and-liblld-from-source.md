## Windows

See https://github.com/zig-lang/zig/wiki/Building-Zig-on-Windows-with-Visual-Studio

## Posix

Typically I use the path `~/local` since it does not require root to install, and it's sandboxed away from the rest of my system. If there's garbage in that directory then I just wipe it and start over.

```
$ cd llvm-5.0.0.src/
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_ASSERTIONS=ON
$ make install
```

```
$ cd cfe-5.0.0.src/
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_ASSERTIONS=ON
$ make install
```

```
$ cd lld-5.0.0.src/
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_ASSERTIONS=ON
$ make install
```

Then add to your zig cmake line that you got from the readme:
`-DCMAKE_PREFIX_PATH=$HOME/local`

If you get stuck you can look at the CI testing scripts for inspiration, keeping in mind that your environment might be different: https://github.com/zig-lang/zig/tree/master/ci

#### Preferring the local, instead of system llvm

If you have a system llvm install then it _may_ possibly be picked up by CMake instead of your local install. Only perform these steps if zig fails to compile at the linking step.

Modify the `cmake/Findllvm.cmake` as follows:

```
# Find the correct local llvm-config executable.
find_program(LLVM_CONFIG_EXE
    NAMES llvm-config5.0 llvm-config
    PATHS
        "~/local/bin"
        "/mingw64/bin"
        "/c/msys64/mingw64/bin"
        "c:/msys64/mingw64/bin"
        "C:/Libraries/llvm-5.0.0/bin"
    NO_DEFAULT_PATH)
```

```
# Do not lookup the local llvm library
#find_library(LLVM_LIBRARY NAMES LLVM)

set(LLVM_LIBRARIES ${LLVM_LIBRARIES} ${LLVM_SYSTEM_LIBS})

#if(LLVM_LIBRARY)
#  set(LLVM_LIBRARIES ${LLVM_LIBRARY})
#endif()
```

Remember to rerun the cmake command and run make again.


#### MacOS

Unfortunately, clang on MacOS does not support `__float128` which the C++ implementation of Zig depends on. The self-hosted Zig compiler supports `f128` on every target, but the self-hosted Zig compiler is not finished yet.

So if you get this error:

```
__float128 is not supported on this target
```

Here's what I did to get Zig installed on MacOS:

```
brew install gcc@7
brew outdated gcc@7 || brew upgrade gcc@7
brew link --overwrite gcc@7
export CC=/usr/local/opt/gcc/bin/gcc-7
export CXX=/usr/local/opt/gcc/bin/g++-7
```

Now follow the 3 instructions about llvm, lld, and clang above. Make sure the `CC` and `CXX` variables are set when you do this.

Next, Zig. Same thing about `CC` and `CXX`.

```
cd zig/
mkdir build
cd build
cmake .. -DCMAKE_PREFIX_PATH=$HOME/local -DCMAKE_INSTALL_PREFIX=$(pwd) -DZIG_LIBC_LIB_DIR=$(dirname $($CC -print-file-name=crt1.o)) -DZIG_LIBC_INCLUDE_DIR=$(echo -n | $CC -E -x c - -v 2>&1 | grep -B1 "End of search list." | head -n1 | cut -c 2- | sed "s/ .*//") -DZIG_LIBC_STATIC_LIB_DIR=$(dirname $($CC -print-file-name=crtbegin.o))
make VERBOSE=1
make install
./zig build --build-file ../build.zig test
```