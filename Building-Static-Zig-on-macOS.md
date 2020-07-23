Related: [[Building Static Zig on FreeBSD]]

On macOS, executables are never fully static; they will always dynamically link against `libSystem` as this is the stable syscall API. Technically, it is possible to create a fully static executable, but such an executable is not guaranteed to continue working with future OS updates. Indeed, Golang-generated macOS executables have suffered exactly this problem due to Go not observing `libSystem` as the official syscall API. So our objective here is to create an executable that is fully self-contained, with the only dynamic library being `libSystem`.

If you run into trouble with the following commands, the first thing to do is to make sure homebrew is upgraded. `brew upgrade`.

```
# Set these variables to whatever you want
export PREFIX=$HOME/local
export TMPDIR=$HOME/tmpz
export LLVMVER="10.0.0"

# I tried using the system default compiler (clang), but it couldn't statically link libc++.
# So we use gcc-9 from homebrew.
export CC=gcc-9
export CXX=g++-9

rm -rf $PREFIX
rm -rf $TMPDIR
mkdir $TMPDIR

cd $TMPDIR
wget https://zlib.net/zlib-1.2.11.tar.xz
tar xf zlib-1.2.11.tar.xz
cd zlib-1.2.11/
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=$PREFIX -DCMAKE_INSTALL_PREFIX=$PREFIX
make install
rm $PREFIX/lib/libz*dylib

cd $TMPDIR
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-$LLVMER/llvm-$LLVMER.src.tar.xz
tar xf llvm-$LLVMVER.src.tar.xz
cd llvm-$LLVMVER.src/
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$PREFIX -DCMAKE_PREFIX_PATH=$PREFIX -DCMAKE_BUILD_TYPE=Release -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD="AVR" -DLLVM_ENABLE_LIBXML2=OFF -DLLVM_ENABLE_TERMINFO=OFF
make install

cd $TMPDIR
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-$LLVMER/lld-$LLVMER.src.tar.xz
tar xf lld-$LLVMVER.src.tar.xz
cd lld-$LLVMVER.src/
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$PREFIX -DCMAKE_PREFIX_PATH=$PREFIX -DCMAKE_BUILD_TYPE=Release
make install

cd $TMPDIR
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-$LLVMER/clang-$LLVMER.src.tar.xz
tar xf clang-$LLVMVER.src.tar.xz
cd clang-$LLVMVER.src/
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$PREFIX -DCMAKE_PREFIX_PATH=$PREFIX -DCMAKE_BUILD_TYPE=Release
make install

cd $TMPDIR
git clone https://github.com/ziglang/zig
cd zig
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=$PREFIX -DCMAKE_INSTALL_PREFIX=$(pwd)/release -DZIG_STATIC=ON
make install
release/bin/zig build docs
cp ../LICENSE release/
cp ../zig-cache/langref.html release/
mv release/bin/zig release/
rmdir release/bin
```

Your static zig installation is in `$TMPDIR/build/release`. To produce a tarball for Continuous Integration:

```
export ARCH="x86_64"
export TARBALLNAME="llvm+clang+lld-$LLVMVER-$ARCH-macosx-gcc9-release"

cd $TMPDIR
mv $PREFIX $TARBALLNAME
tar cfJ $TARBALLNAME.tar.xz $TARBALLNAME/
s3cmd put -P --no-mime-magic --add-header="cache-control: public, max-age=31536000, immutable" $TARBALLNAME.tar.xz s3://ziglang.org/builds/
```