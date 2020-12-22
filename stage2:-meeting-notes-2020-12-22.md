# Agenda

1. @andrewrk will demonstrate `tracy` integration -- `tracy` is a performance profiling tool [add a link to Andrew showcasing it in a live coding session]
2. How will we handle `threadlocal` as a keyword? How is it currently handled in stage1 with LLVM?
3. What is missing in stage2 to get the standard basic binary like

```zig
const std = @import("std");

pub fn main() void {
    std.log.info("Hello, world!, .{});
}
```

to successfully compile?

# Meeting notes