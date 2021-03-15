The process-headers tool is not needed for these headers because mingw-w64 headers are already multi-architecture.

```sh
git clone git://git.code.sf.net/p/mingw-w64/mingw-w64
```

Check out the latest release.

```sh
ZIGSRC=/path/to/zig/src/tree
rm -rf $ZIGSRC/lib/libc/include/any-windows-any
cd mingw-w64-headers
./configure --prefix=/tmp/prefix --with-default-win32-winnt=0x0603
make install
mv /tmp/prefix/include $ZIGSRC/lib/libc/include/any-windows-any
```

The above makes Windows 8.1 the minimum version. Once Windows 10 is the minimum supported version (on [January 10, 2023](https://support.microsoft.com/en-us/help/13853/windows-lifecycle-fact-sheet)), we can additionally pass `--with-default-msvcrt=ucrt`.

Next, update all the files in `lib/libc/mingw/*`.

Examine carefully the diff between `v7.0.0` and the version that you are updating to, and consider if anything else needs to be done. Update the previous sentence in this wiki article to the version you updated to.
