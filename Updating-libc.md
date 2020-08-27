Here's how to update the libc files that Zig bundles.

## glibc

Make sure these dependencies are installed. If the scripts below fail, I'm not aware of a way to make them resume; each command deletes everything and starts over.

 * python3
 * subversion
 * gawk
 * autotools
 * flex
 * bison
 * build-essential
 * texinfo

Next, git clone glibc.

```
git clone git://sourceware.org/git/glibc.git
cd glibc
git checkout glibc-2.32 # the tag of the version to update to
```

Assuming the path of that is `~/glibc`, make a new directory and go to it. Then run the Python commands, each of which uses all CPU cores and takes a long time. If any of them fail (except for the csky one), look at the logs to find out why, correct it, and then start the command again. Unfortunately each command will delete its own previous progress and start over.

```
mkdir multi
cd multi
python3 ~/glibc/scripts/build-many-glibcs.py . checkout
cd src/glibc
git checkout glibc-2.32 # the tag of the version to update to
cd -
python3 ~/glibc/scripts/build-many-glibcs.py . host-libraries
python3 ~/glibc/scripts/build-many-glibcs.py . compilers # takes upwards of 12 hours with 16 CPU cores, might want to run overnight
python3 ~/glibc/scripts/build-many-glibcs.py . glibcs # took 7 hours for me with 8 CPU cores
```

Next, make sure that the list of architectures in `tools/process_headers.zig` is complete in that it lists all of the glibc targets (except csky) and maps them to Zig targets. Any additional targets you add, add to the `libcs_available` variable in `target.cpp`.

Next, from the "build" directory of zig git source, use `tools/process_headers.zig`:

```
./zig build-exe ../tools/process_headers.zig
./process_headers --search-path ~/glibc/multi/install/glibcs --out hdrs --abi glibc
```

Inspect the `hdrs` directory that the tool just created. If it looks good, then:

```
rm -rf $(find ../lib/libc/include/ -name "*linux-gnu*")
rm -rf ../lib/libc/include/generic-glibc
mv hdrs/* ../lib/libc/include/
```

Inspect `git status` and make sure the changes look good. Make a commit with only the updated glibc headers in it.

Next, use `tools/update_glibc.zig`.

```
./zig build-exe ../tools/update_glibc.zig
./update_glibc ~/Downloads/glibc ../lib
```

This should update `abi.txt`, `fns.txt`, and `vers.txt`.

If you keep your glibc build artifacts, you can use it with `zig build test -Denable-qemu -Denable-foreign-glibc=/foo/glibc/multi/install/glibcs`.

## musl

```
git clone git://git.musl-libc.org/musl
git checkout v1.2.0 # the tag of the version to update to
rm -rf obj/ && make DESTDIR=build-all/aarch64 install-headers ARCH=aarch64
rm -rf obj/ && make DESTDIR=build-all/arm install-headers ARCH=arm
rm -rf obj/ && make DESTDIR=build-all/i386 install-headers ARCH=i386
rm -rf obj/ && make DESTDIR=build-all/mips install-headers ARCH=mips
rm -rf obj/ && make DESTDIR=build-all/mips64 install-headers ARCH=mips64
rm -rf obj/ && make DESTDIR=build-all/powerpc install-headers ARCH=powerpc
rm -rf obj/ && make DESTDIR=build-all/powerpc64 install-headers ARCH=powerpc64
rm -rf obj/ && make DESTDIR=build-all/riscv64 install-headers ARCH=riscv64
rm -rf obj/ && make DESTDIR=build-all/s390x install-headers ARCH=s390x
rm -rf obj/ && make DESTDIR=build-all/x86_64 install-headers ARCH=x86_64
```

Make sure the list of architectures in `tools/process_headers.zig` is complete in that it lists all of the musl targets which have corresponding Zig targets. Any additional targets you add, add to the `libcs_available` variable in `target.cpp`.

Next, use `tools/process_headers.zig`, with these parameters:
 * `--abi musl`
 * `--search-path` parameter will be the path to the `build-all` directory.
 * `--out` to a temporary directory if you want to inspect the output first, or `lib/libc/include` if you are confident in the output.

To update musl source code:

```
cd lib/libc/musl
rm -rf arch crt compat src include
cp -r ~/Downloads/musl/arch ./
cp -r ~/Downloads/musl/crt ./
cp -r ~/Downloads/musl/compat ./
cp -r ~/Downloads/musl/src ./
cp -r ~/Downloads/musl/include ./
```

Remove the non-supported architectures:

```
rm -rf arch/m68k
rm -rf arch/microblaze
rm -rf arch/mipsn32
rm -rf arch/or1k
rm -rf arch/sh
rm -rf arch/x32
```

Next, look at a `git status` and `git diff` and make sure everything looks OK. There are some chores to do below:

Update the contents of `libc/musl/src/internal/version.h` to the correct musl version number. This file will have been deleted by the above process and you will need to create it now (look at the `git diff` to see the contents).

Take note of the `.mak` files. These will show up in the "Untracked files" section, and should be deleted:

```
rm arch/arm/arch.mak
rm arch/i386/arch.mak
rm arch/mips/arch.mak
rm arch/powerpc/arch.mak
```

If there are any new ones not covered in this list, support needs to be added in link.cpp, where there is special handling for these. Look for `time32_compat_arch_list`.

Update `ZIG_MUSL_SRC_FILES` in `src/install_files.h` to be a complete list, e.g. with `find musl/src -type f -name "*.c" -o -iname "*.s"`. Similarly, update `ZIG_MUSL_COMPAT_TIME32_FILES`, e.g. with `find musl/compat/time32 -type f -name "*.c" -o -iname "*.s"`.

If musl added any new architectures, add them to `musl_arch_names` in `link.cpp`. These can be found by `ls arch/` in the musl source directory.

## freebsd

TODO

## openbsd

TODO

## netbsd

TODO

## mingw-w64 (Windows)

The process-headers tool is not needed for these headers because mingw-w64 headers are already multi-architecture.

```
git clone git://git.code.sf.net/p/mingw-w64/mingw-w64
```

Check out the latest release.

```
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
