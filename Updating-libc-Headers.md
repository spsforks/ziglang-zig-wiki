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

Next, make sure that the list of architectures in `libc/process_headers.zig` is complete in that it lists all of the glibc targets (except csky) and maps them to Zig targets. Any additional targets you add, add to the `libcs_available` variable in `target.cpp`.

### musl

Download all the "native" tarballs from http://musl.cc/.

Untar all of them.

Delete all the `c++` directories, e.g. `rm -rf $(find . -name "c++" -type d)`.

Delete the files that have case conflicts. For example:

```
 libc/include/generic-musl/linux/netfilter/xt_CONNMARK.h  |  7 -------
 libc/include/generic-musl/linux/netfilter/xt_DSCP.h      | 27 ---------------------------
 libc/include/generic-musl/linux/netfilter/xt_MARK.h      |  7 -------
 libc/include/generic-musl/linux/netfilter/xt_RATEEST.h   | 17 -----------------
 libc/include/generic-musl/linux/netfilter/xt_TCPMSS.h    | 13 -------------
 libc/include/generic-musl/linux/netfilter_ipv4/ipt_ECN.h | 34 ----------------------------------
 libc/include/generic-musl/linux/netfilter_ipv4/ipt_TTL.h | 24 ------------------------
 libc/include/generic-musl/linux/netfilter_ipv6/ip6t_HL.h | 25 -------------------------
```

Make sure the list of architectures in `libc/process_headers.zig` is complete in that it lists all of the musl targets which have corresponding Zig targets. Any additional targets you add, add to the `libcs_available` variable in `target.cpp`.

When updating the actual musl C source code, replace `#include` directives that have relative lookups like this with `#include_next`:

```
$ grep -RI '\.\.\/\.\.\/'
src/include/errno.h:#include "../../include/errno.h"
src/include/pthread.h:#include "../../include/pthread.h"
src/include/stdio.h:#include "../../include/stdio.h"
src/include/arpa/inet.h:#include "../../../include/arpa/inet.h"
src/include/stdlib.h:#include "../../include/stdlib.h"
src/include/time.h:#include "../../include/time.h"
src/include/unistd.h:#include "../../include/unistd.h"
src/include/crypt.h:#include "../../include/crypt.h"
src/include/langinfo.h:#include "../../include/langinfo.h"
src/include/string.h:#include "../../include/string.h"
src/include/signal.h:#include "../../include/signal.h"
src/include/sys/mman.h:#include "../../../include/sys/mman.h"
src/include/sys/auxv.h:#include "../../../include/sys/auxv.h"
src/include/sys/time.h:#include "../../../include/sys/time.h"
src/include/sys/sysinfo.h:#include "../../../include/sys/sysinfo.h"
src/include/features.h:#include "../../include/features.h"
src/include/resolv.h:#include "../../include/resolv.h"
```

This is so that we don't need an additional copy of the libc headers.

Update the contents of `libc/musl/src/internal/version.h` to the correct musl version number.

### freebsd

TODO

### openbsd

TODO

### netbsd

TODO

## Use the process_headers tool

In the Zig source repo, use the process_headers tool.

```
zig run libc/process_headers.zig --help
```

This tool will create a headers directory that contains a `generic` subdir as well as architecture subdirs.

For glibc, when you do a git diff and look at the updated headers, it will have deleted a bunch of `asm/unistd.h` files. This is because I did those manually the first time. You'll have to go back and edit process_headers.zig to patch in the Linux headers for glibc, since it depends on them, and then update this wiki page.

## Windows

Windows can be updated independently from the others since the process-headers tool does not accomplish anything for these headers.

```
git clone git://git.osdn.net/gitroot/mingw/mingw-org-wsl.git
```

Check out the latest release.

Copy `w32api/include/*` to `zig/libc/include/any-windows-any/*`.

Copy `mingwrt/include/*` to `zig/libc/include/generic-mingw/*`. Rename `zig/libc/include/generic-mingw/_mingw.h.in` to `zig/libc/include/generic-mingw/_mingw.h`.
