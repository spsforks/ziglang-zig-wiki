```sh
FREEBSD_VERSION='release/12.2.0'
```

## download the freebsd repo and checkout to the new version
```sh
git clone https://git.freebsd.org/src.git freebsd
cd freebsd
git checkout $FREEBSD_VERSION
cd -
```

## update include headers
```sh
cp -R ~/freebsd/include/* ~/zig/lib/libc/include/any-freebsd-any
cp -R ~/freebsd/sys/sys/* ~/zig/lib/libc/include/any-freebsd-any/sys

cp -R ~/freebsd/sys/amd64/include/* ~/zig/lib/libc/include/x86_64-freebsd-any/machine
cp -R ~/freebsd/sys/x86/include/* ~/zig/lib/libc/include/x86_64-freebsd-any/x86

rm ~/zig/lib/libc/include/any-freebsd-any/mk-osreldate.h
```

## update `.c` sources for static builds