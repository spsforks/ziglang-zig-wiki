Related: [[Building Static Zig on macOS]]

On FreeBSD, proper executables are never fully static; they always dynamically link against system libc as this is the stable syscall API. Technically, it is possible to create a fully static executable, but such an executable is not guaranteed to continue working with future OS updates. So our objective here is to create an executable that is fully self-contained, with the only dynamic library being system libc.

```
# Setup
pkg install cmake

# Set these variables to whatever you want
export LLVMVER="11.0.0"
export ARCH="x86_64"
export TARBALLNAME="llvm+clang+lld-$LLVMVER-$ARCH-freebsd-release"
export TMPDIR=$HOME/tmpz
export PREFIX=$TMPDIR/$TARBALLNAME

rm -rf $PREFIX
rm -rf $TMPDIR
mkdir $TMPDIR

# Install LLVM
cd $TMPDIR
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-$LLVMVER/llvm-$LLVMVER.src.tar.xz
tar xf llvm-$LLVMVER.src.tar.xz
cd llvm-$LLVMVER.src/
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$PREFIX -DCMAKE_PREFIX_PATH=$PREFIX -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_LIBXML2=OFF -DLLVM_ENABLE_TERMINFO=OFF
make install

# Install LLD
cd $TMPDIR
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-$LLVMVER/lld-$LLVMVER.src.tar.xz
tar xf lld-$LLVMVER.src.tar.xz
cd lld-$LLVMVER.src/
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$PREFIX -DCMAKE_PREFIX_PATH=$PREFIX -DCMAKE_BUILD_TYPE=Release
make install

# Install Clang
cd $TMPDIR
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-$LLVMVER/clang-$LLVMVER.src.tar.xz
tar xf clang-$LLVMVER.src.tar.xz
cd clang-$LLVMVER.src/
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$PREFIX -DCMAKE_PREFIX_PATH=$PREFIX -DCMAKE_BUILD_TYPE=Release
make install

# Install Zig
pkg install git
cd $TMPDIR
git clone https://github.com/ziglang/zig
cd zig
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=$PREFIX -DCMAKE_INSTALL_PREFIX=$(pwd)/release -DZIG_STATIC=ON
make install
release/bin/zig build docs
```

To produce a tarball for Continuous Integration:

```
cd $TMPDIR
pkg install py27-s3cmd
s3cmd --configure
tar cfJ $TARBALLNAME.tar.xz $TARBALLNAME/
s3cmd put -P --no-mime-magic --add-header="cache-control: public, max-age=31536000, immutable" $TARBALLNAME.tar.xz s3://ziglang.org/deps/
```