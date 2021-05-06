This is a collaborative effort to note quick answers to questions about programming in Zig that get asked frequently by newcomers.  
If you are about to answer a question for the third time in a row, write it here and then paste a link to cache it for the future :)  
_(maybe start by considering if it wouldn't be better to just edit an existing answer)_

## How do I parse a number from a string?
Use [`std.fmt.parseInt`](https://github.com/ziglang/zig/blob/3bf72f2b3add70ad0671c668baf251f2db93abbf/lib/std/fmt.zig#L1357) or [`std.fmt.parseFloat`](https://github.com/ziglang/zig/blob/3bf72f2b3add70ad0671c668baf251f2db93abbf/lib/std/fmt/parse_float.zig#L341).

## How do I do x with strings?
In zig, strings are just bytes, so the stdlib namespace you want to look at is `std.mem`, it has functions for concatenation, substring replacement, comparison, etc. the other namespace of interest to you will be `std.fmt` for formatting functions.

## github submodule cmake example

```zig
const Builder = @import("std").build.Builder;

pub fn build(b: *Builder) !void {

    const mode = b.standardReleaseOptions();
    const target = b.standardTargetOptions(.{});
    
    const DOtherSide_prebuild = b.addSystemCommand(&[_][]const u8{
        "cmake",
        "-B",
        "deps/dotherside/build",
        "-S",
        "deps/dotherside",
        "-DCMAKE_BUILD_TYPE=Release"
    });
    try DOtherSide_prebuild.step.make();
    const DOtherSide_build = b.addSystemCommand(&[_][]const u8{
        "cmake",
        "--build",
        "deps/dotherside/build",
        "-j"
    });
    try DOtherSide_build.step.make();
    
    const exe = b.addExecutable("qml_zig", "src/main.zig");
    exe.setBuildMode(mode);
    exe.setTarget(target);
    exe.addLibPath("deps/dotherside/build/lib");
    exe.linkSystemLibrary("DOtherSide");
    exe.install();
 // ....
}
```

##  Benefits of specifying the error set at the function return type, provide any benefits?

Think of the two function declarations,

```
fn createFile() !void { 

fn failFnCounter() error{Oops}!void { 
```

it makes a library much easier to handle because you can just look in one (or a few) places for all errors the function will return.

Explicit error sets are good for API boundaries. 

It ensures,
1) consumers can easily see all your errors, and 
2) you don't accidentally let internal errors escape
