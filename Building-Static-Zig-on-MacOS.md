On MacOS, executables are never fully static; they will always dynamically link against `libSystem` as this is the stable syscall API. Technically, it is possible to create a fully static executable, but such an executable is not guaranteed to continue working with future OS updates. Indeed, Golang-generated MacOS executables have suffered exactly this problem due to Go not observing `libSystem` as the official syscall API. So our objective here is to create an executable that is fully self-contained, with the only dynamic library being `libSystem`.

```
# Set these two variables to whatever you want
export PREFIX=$HOME/local
export TMPDIR=$HOME/tmpz

# I tried using the system default compiler (clang), but it couldn't statically link libc++.
# So we use gcc-7 from homebrew.
export CC=gcc-7
export CXX=g++-7

rm -rf $PREFIX
rm -rf $TMPDIR
mkdir $TMPDIR

cd $HOME/tmpz
wget https://zlib.net/zlib-1.2.11.tar.xz
tar xf zlib-1.2.11.tar.xz
cd zlib-1.2.11/
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=$PREFIX -DCMAKE_INSTALL_PREFIX=$PREFIX
make install
rm $PREFIX/lib/libz*dylib

cd $HOME/tmpz
wget ftp://ftp.invisible-island.net/ncurses/ncurses.tar.gz
tar xf ncurses.tar.gz
cd ncurses-6.1/
./configure --without-shared --prefix=$PREFIX
make install

cd $HOME/tmpz
wget http://releases.llvm.org/7.0.0/llvm-7.0.0.src.tar.xz
tar xf llvm-7.0.0.src.tar.xz
cd llvm-7.0.0.src/
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$PREFIX -DCMAKE_PREFIX_PATH=$PREFIX -DCMAKE_BUILD_TYPE=Release -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD="WebAssembly;AVR;RISCV" -DLLVM_ENABLE_LIBXML2=OFF
make install

cd $HOME/tmpz
wget http://releases.llvm.org/7.0.0/cfe-7.0.0.src.tar.xz
tar xf cfe-7.0.0.src.tar.xz
cd cfe-7.0.0.src/
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$PREFIX -DCMAKE_PREFIX_PATH=$PREFIX -DCMAKE_BUILD_TYPE=Release
make install

cd $HOME/tmpz
git clone https://github.com/ziglang/zig
cd zig
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=$PREFIX -DCMAKE_INSTALL_PREFIX=$(pwd)/release -DZIG_STATIC=ON
make install
release/bin/zig build --build-file ../build.zig docs
cp ../LICENSE release/
cp ../zig-cache/langref.html release/
mv release/bin/zig release/
rmdir release/bin
```

Your static zig installation is in `$HOME/tmpz/build/release`.