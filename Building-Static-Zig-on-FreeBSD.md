Related: [[Building Static Zig on macOS]]

On FreeBSD, proper executables are never fully static; they always dynamically link against system libc as this is the stable syscall API. Technically, it is possible to create a fully static executable, but such an executable is not guaranteed to continue working with future OS updates. So our objective here is to create an executable that is fully self-contained, with the only dynamic library being system libc.

```
# Setup
pkg install cmake

# Set these two variables to whatever you want
export PREFIX=$HOME/local
export TMPDIR=$HOME/tmpz

rm -rf $PREFIX
rm -rf $TMPDIR
mkdir $TMPDIR

# Install LLVM
cd $TMPDIR
wget https://releases.llvm.org/9.0.0/llvm-9.0.0.src.tar.xz
tar xf llvm-9.0.0.src.tar.xz
cd llvm-9.0.0.src/
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$PREFIX -DCMAKE_PREFIX_PATH=$PREFIX -DCMAKE_BUILD_TYPE=Release -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD="AVR" -DLLVM_ENABLE_LIBXML2=OFF -DLLVM_ENABLE_TERMINFO=OFF
make install

# Install Clang
cd $TMPDIR
wget https://releases.llvm.org/9.0.0/cfe-9.0.0.src.tar.xz
tar xf cfe-9.0.0.src.tar.xz
cd cfe-9.0.0.src/
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
cd
pkg install py27-s3cmd
s3cmd --configure
mv $PREFIX llvm+clang-9.0.0-freebsd-x86_64-release
tar cfJ llvm+clang-9.0.0-freebsd-x86_64-release.tar.xz llvm+clang-9.0.0-freebsd-x86_64-release/
s3cmd put -P llvm+clang-9.0.0-freebsd-x86_64-release.tar.xz s3://ziglang.org/builds/
```