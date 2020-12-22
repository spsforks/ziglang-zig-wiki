# Agenda

1. @andrewrk will demonstrate `tracy` integration -- `tracy` is a performance profiling tool. Here's a link to Andrew's old live coding session about it: [youtube.com](https://youtu.be/c0LRjqgtKS0).
2. @kubkon asked how we will handle `threadlocal` as a keyword. How is it currently handled in stage1 with LLVM?
3. @kubkon asked what is currently missing in stage2 to get the standard basic binary like

```zig
const std = @import("std");

pub fn main() void {
    std.log.info("Hello, world!, .{});
}
```

to successfully compile?

4. @andrewrk will discuss the new thread pool landing in `master`: [ziglang/zig#7462](https://github.com/ziglang/zig/pull/7462).
5. @kubkon made a couple of major changes to how we handle Mach-O in stage2 in the sense that, much like for Elf, we preallocate space for various segments and sections to aid in incremental linking. This allows us to skip rewriting `__LINKEDIT` segment every single time `__text` changes for instance. However, this means we break out from the convention set by other tools such as Apple's `ld` or LLVM's `ld64`. Additionally, this may imply that Apple's `codesign` tool will not validate the Mach-O binary due to certain hard-coded (wrong) assumptions about the structure of the Mach-O. Question here is: should we leave it as-is for Debug builds, and perhaps follow the convention in Release?

# Meeting notes