For macOS, we only need to fetch the latest libc headers. The easiest way to achieve this is to use
[fetch-them-macos-headers](https://github.com/ziglang/fetch-them-macos-headers) utility. The only requirement
is that you have to run it directly on the target (i.e., a native macOS installation).

```sh
git clone https://github.com/kubkon/fetch-them-macos-headers
cd fetch-them-macos-headers
zig build run
```

This will create a new dir `x86_64-macos-gnu` in cwd with all the required headers for macOS. Assuming that
Zig source is in `~/zig`, simply copy the dir over

```sh
rm -rf ~/zig/lib/libc/include/x86_64-macos-gnu
mv x86_64-macos-gnu ~/zig/lib/libc/include/.
```
