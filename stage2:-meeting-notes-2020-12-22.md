# Agenda

1. @andrewrk will demonstrate `tracy` integration -- `tracy` is a performance profiling tool. Here's a link to Andrew's old live coding session about it: [youtube.com](https://youtu.be/c0LRjqgtKS0).
2. @kubkon asked how we will handle `threadlocal` as a keyword. How is it currently handled in stage1 with LLVM?
3. @kubkon asked what is currently missing in stage2 to get the standard basic binary like

```zig
const std = @import("std");

pub fn main() void {
    std.log.info("Hello, world!", .{});
}
```

to successfully compile?

4. @andrewrk will discuss the new thread pool landing in `master`: [ziglang/zig#7462](https://github.com/ziglang/zig/pull/7462).
5. @kubkon made a couple of major changes to how we handle Mach-O in stage2 in the sense that, much like for Elf, we preallocate space for various segments and sections to aid in incremental linking. This allows us to skip rewriting `__LINKEDIT` segment every single time `__text` changes for instance. However, this means we break out from the convention set by other tools such as Apple's `ld` or LLVM's `ld64`. Additionally, this may imply that Apple's `codesign` tool will not validate the Mach-O binary due to certain hard-coded (wrong) assumptions about the structure of the Mach-O. Question here is: should we leave it as-is for Debug builds, and perhaps follow the convention in Release?

# Meeting notes

1. A command that might be useful for Unix-like OSes:

```
nix-shell  -p cmake -p ninja -p pkg-config -p glfw -p freetype -p capstone -p tbb -p gtk3-x11
```

2. Thread-local is delegated to an appropriate LLVM API function. @kubkon demonstrated the origin of the question being a failing libstd test on `aarch64-macos-gnu` to do with declaring a global variable as thread-local. After some live debugging, we concluded that both `clang` and `zig` generate almost identical LLVM IR for it, and @kubkon later (offline) verified that when we swap LLD for Apple's system linker, the problem disappears. @andrewrk also mentioned that he posted a question on LLVM-dev mailing list inquiring what the chances are of `aarch64` MachO2 backend being ready for LLVM 12 release, and got a disheartening reply stating that "it's ready when it's ready". So yet another point in favour of ditching LLD and putting more effort into our own linker(s).

3. @andrewrk had a stab at answering that one and the answer is that we've got quite a few major hurdles to go over before that's done
   including `comptime`, panic handlers, etc. Specifically for Mach-O however, a good first achievable milestone would be to add support for
   `extern` in stage2 and link with `libSystem` functions instead of hand-writing `print` and `exit` in pure assembly as it is done now.
   
   EDIT: the downstream tracking issue: [ziglang/zig#7527](https://github.com/ziglang/zig/issues/7527)

4. TODO @andrewrk could you write this one up as I AFK when you were discussing it?

5. The entire point boils down to whether we should worry about the generated Mach-O binaries be compatible with Apple's `codesign` tool, and the short answer is no. @andrewrk suggested that we implement the incremental linker in the most optimal and yet correct way (which BTW is not what Apple assumes in their `codesign` or more specifically `codesign_allocate` tool), and see if our users are OK with it, and perhaps, with a bit of luck, have Apple fix their incorrect assumptions in `codesign_allocate.