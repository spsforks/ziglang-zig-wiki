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
