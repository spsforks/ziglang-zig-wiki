Typically I use the path `~/local` since it does not require root to install, and it's sandboxed away from the rest of my system. If there's garbage in that directory then I just wipe it and start over.

```
$ cd llvm-4.0.0.src/
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local -DCMAKE_BUILD_TYPE=RelWithDebInfo
$ make install
```

```
$ cd cfe-4.0.0.src/
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local -DCMAKE_BUILD_TYPE=RelWithDebInfo
$ make install
```

```
$ cd lld-4.0.0.src/
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local -DCMAKE_BUILD_TYPE=RelWithDebInfo
$ make install
```

Then add to your zig cmake line that you got from the readme:
`-DCMAKE_PREFIX_PATH=$HOME/local`
