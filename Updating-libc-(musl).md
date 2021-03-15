```sh
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

```sh
cd lib/libc/musl
rm -rf arch crt compat src include
cp -r ~/Downloads/musl/arch ./
cp -r ~/Downloads/musl/crt ./
cp -r ~/Downloads/musl/compat ./
cp -r ~/Downloads/musl/src ./
cp -r ~/Downloads/musl/include ./
```

Remove the non-supported architectures:

```sh
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

```sh
rm arch/arm/arch.mak
rm arch/i386/arch.mak
rm arch/mips/arch.mak
rm arch/powerpc/arch.mak
```

If there are any new ones not covered in this list, support needs to be added in link.cpp, where there is special handling for these. Look for `time32_compat_arch_list`.

Update `src_files` in `src/musl.zig` to be a complete list, e.g. with `find musl/src -type f -name "*.c" -o -iname "*.s"`. Similarly, update `compat_time32_files`, e.g. with `find musl/compat/time32 -type f -name "*.c" -o -iname "*.s"`.

If musl added any new architectures, add them to `musl_arch_names` in `src/musl.zig`. These can be found by `ls arch/` in the musl source directory.

To update the `lib/libc/musl/libc.s` file containing stubs for all the dynamic symbols of musl's `libc.so` first build musl normally by running `make` in the root of the musl repository. Then navigate to the root of the zig source repository and run the following commands:

```sh
zig build-exe tools/gen_stubs.zig
objdump --dynamic-syms /path/to/musl/lib/libc.so | ./gen_stubs > lib/libc/musl/libc.s
```

check the `git diff` to make sure everything seems sane.

To verify that the stub `libc.so` matches the "real" musl `libc.so` first build a zig hello world that dynamically links musl:

```sh
zig build-exe -lc -dynamic -target x86_64-linux-musl hello.zig --verbose-link
```

Copy the path to the stub `libc.so` in the global cache from the link command logged and then run the following commands (with the proper paths):

```sh
objdump --dynamic-syms /home/ifreund/.cache/zig/o/94cd8ea1da001f912f2f7d259446424d/libc.so | sed -E -e 's/[0-9a-f]{16}//g' | sort > stubs.txt
objdump --dynamic-syms /path/to/musl/lib/libc.so | sed -E -e 's/[0-9a-f]{16}//g' | sort > musl.txt                                                
diff musl.txt stubs.txt
```

If all is well, the only output of the diff command should be the line containing the filename. For example:

```bash session
1702c1702
< ../musl/lib/libc.so:     file format elf64-x86-64
---
> /home/ifreund/.cache/zig/o//libc.so:     file format elf64-x86-64
```
