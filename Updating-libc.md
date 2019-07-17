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
```

Assuming the path of that is `~/glibc`, make a new directory and go to it. Then run the Python commands, each of which uses all CPU cores and takes a long time. If any of them fail (except for the csky one), look at the logs to find out why, correct it, and then start the command again. Unfortunately each command will delete its own previous progress and start over.

```
mkdir multi
cd multi
python3 ~/glibc/scripts/build-many-glibcs.py . checkout
python3 ~/glibc/scripts/build-many-glibcs.py . host-libraries
python3 ~/glibc/scripts/build-many-glibcs.py . compilers
python3 ~/glibc/scripts/build-many-glibcs.py . glibcs
```

Next, make sure that the list of architectures in `libc/process_headers.zig` is complete in that it lists all of the glibc targets (except csky) and maps them to Zig targets. Any additional targets you add, add to the `libcs_available` variable in `target.cpp`.

Next, proceed to [using the process_headers tool](#using-the-process_headers-tool).

## musl

```
git clone git://git.musl-libc.org/musl
git checkout v1.1.23 # the tag of the version to update to
make DESTDIR=build-all/aarch64 install-headers ARCH=aarch64
make DESTDIR=build-all/arm install-headers ARCH=arm
make DESTDIR=build-all/i386 install-headers ARCH=i386
make DESTDIR=build-all/mips install-headers ARCH=mips
make DESTDIR=build-all/mips64 install-headers ARCH=mips64
make DESTDIR=build-all/powerpc install-headers ARCH=powerpc
make DESTDIR=build-all/powerpc64 install-headers ARCH=powerpc64
make DESTDIR=build-all/riscv64 install-headers ARCH=riscv64
make DESTDIR=build-all/s390x install-headers ARCH=s390x
make DESTDIR=build-all/x86_64 install-headers ARCH=x86_64
```

Make sure the list of architectures in `libc/process_headers.zig` is complete in that it lists all of the musl targets which have corresponding Zig targets. Any additional targets you add, add to the `libcs_available` variable in `target.cpp`.

Next, [use the process_headers tool](#using-the-process_headers-tool), with these parameters:
 * `--abi musl`
 * `--search-path` parameter will be the path to the `build-all` directory.
 * `--out` to a temporary directory if you want to inspect the output first, or `lib/libc/include` if you are confident in the output.

To update musl source code:

```
cd lib/libc/musl/arch 
rm -rf arch crt src include
cp -r ~/downloads/musl/arch ./
cp -r ~/downloads/musl/crt ./
cp -r ~/downloads/musl/src ./
cp -r ~/downloads/musl/include ./
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

Next, look at a `git diff` and make sure everything looks OK.

Update the contents of `libc/musl/src/internal/version.h` to the correct musl version number. This file will have been deleted by the above process and you will need to create it now (look at the `git diff` to see the contents).

Update `ZIG_MUSL_SRC_FILES` in `src/install_files.h` to be a complete list, e.g. with `find musl/src -type f`.

If musl added any new architectures, add them to `musl_arch_names` in `link.cpp`. These can be found by `ls arch/` in the musl source directory.

## freebsd

TODO

## openbsd

TODO

## netbsd

TODO

## Using the process_headers tool

In the Zig source repo, use the process_headers tool.

```
zig run tools/process_headers.zig --help
```

This tool will create a headers directory that contains a `generic` subdir as well as architecture subdirs.

For glibc, when you do a git diff and look at the updated headers, it will have deleted a bunch of `asm/unistd.h` files. This is because I did those manually the first time. You'll have to go back and edit process_headers.zig to patch in the Linux headers for glibc, since it depends on them, and then update this wiki page.

## mingw-w64 (Windows)

The process-headers tool is not needed for these headers because mingw-w64 headers are already multi-architecture.

```
git clone git://git.code.sf.net/p/mingw-w64/mingw-w64
```

Check out the latest release.

```
cd mingw-w64-headers
./configure --prefix=/path/to/zig/libc/include/any-windows-any --with-default-win32-winnt=0x0601
make install
```

The above makes Windows 7 the minimum version. Once Windows 10 is the minimum supported version, we can additionally pass `--with-default-msvcrt=ucrt`.

Next, update all the files in `lib/libc/mingw/*`.

Examine carefully the diff between `v6.0.0` and the version that you are updating to, and consider if anything else needs to be done. Update the previous sentence in this wiki article to the version you updated to.
