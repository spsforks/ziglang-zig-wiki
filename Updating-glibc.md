> Note for next time: llvm 12 gained csky support, so these instructions will likely need to be modified to take that into account.

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

```sh
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

```sh
./zig build-exe ../tools/process_headers.zig
./process_headers --search-path ~/glibc/multi/install/glibcs --out hdrs --abi glibc
```

Inspect the `hdrs` directory that the tool just created. If it looks good, then:

```sh
rm -rf $(find ../lib/libc/include/ -name "*linux-gnu*")
rm -rf ../lib/libc/include/generic-glibc
mv hdrs/* ../lib/libc/include/
```

Inspect `git status` and make sure the changes look good. Make a commit with only the updated glibc headers in it.

Next, use `tools/update_glibc.zig`.

```sh
./zig build-exe ../tools/update_glibc.zig
./update_glibc ~/Downloads/glibc ../lib
```

This should update `abi.txt`, `fns.txt`, and `vers.txt`.

If you keep your glibc build artifacts, you can use it with `zig build test -Denable-qemu -Denable-foreign-glibc=/foo/glibc/multi/install/glibcs`.
