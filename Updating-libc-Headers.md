Here's how to update the libc header files that Zig bundles.

## Create Builds of libc for Every Architecture

### glibc

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

Next, make sure that the list of architectures in `libc/process_headers.zig` is complete in that it lists all of the glibc targets (except csky) and maps them to Zig targets.

### musl

Download all the "native" tarballs from http://musl.cc/.

Untar all of them.

Delete all the `c++` directories, e.g. `rm -rf $(find . -name "c++" -type d)`.

Make sure the list of architectures in `libc/process_headers.zig` is complete in that it lists all of the musl targets which have corresponding Zig targets.

### freebsd

TODO

### openbsd

TODO

### netbsd

TODO

### Windows

Determine which libc headers to use. They should come with Windows.h hopefully. Then document the process here.

## Use the process_headers tool

In the Zig source repo, use the process_headers tool.

```
zig run libc/process_headers.zig --help
```

This tool will create a headers directory that contains a `generic` subdir as well as architecture subdirs.

For glibc, when you do a git diff and look at the updated headers, it will have deleted a bunch of `asm/unistd.h` files. This is because I did those manually the first time. You'll have to go back and edit process_headers.zig to patch in the Linux headers for glibc, since it depends on them, and then update this wiki page.
