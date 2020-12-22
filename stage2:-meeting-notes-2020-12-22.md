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

# Meeting notes