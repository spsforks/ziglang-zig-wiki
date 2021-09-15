This is a collaborative effort to note quick answers to questions about programming in Zig that get asked frequently by newcomers.  
If you are about to answer a question for the third time in a row, write it here and then paste a link to cache it for the future :)  
_(maybe start by considering if it wouldn't be better to just edit an existing answer)_

## How do I parse a number from a string?
Use [`std.fmt.parseInt`](https://github.com/ziglang/zig/blob/3bf72f2b3add70ad0671c668baf251f2db93abbf/lib/std/fmt.zig#L1357) or [`std.fmt.parseFloat`](https://github.com/ziglang/zig/blob/3bf72f2b3add70ad0671c668baf251f2db93abbf/lib/std/fmt/parse_float.zig#L341).

## How do I do x with strings?
In zig, strings are just bytes, so the stdlib namespace you want to look at is `std.mem`, it has functions for concatenation, substring replacement, comparison, etc. the other namespace of interest to you will be `std.fmt` for formatting functions.

## How do I read input one line at a time?

First, obtain a `Reader` for the input you're processing. For example, for `std.fs.File` you can obtain a `std.fs.File.Reader` by calling the `.reader()` method.

Then, use [`readUntilDelimiterOrEof()`](https://ziglang.org/documentation/master/std/#std;fs.File.Reader.readUntilDelimiterOrEof) or [`readUntilDelimiterOrEofAlloc()`](https://ziglang.org/documentation/master/std/#std;fs.File.Reader.readUntilDelimiterOrEofAlloc):

```zig
var buffer: [1024]u8 = undefined;
while (try reader.readUntilDelimiterOrEof(&buffer, '\n')) |line| {
  // do something with 'line'
}
```

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


## What is a `[*]T`?

It's a many-pointer (a type of pointer). [a.k.a multi-pointer.]  
"A slice without a length;" it can be indexed into or sliced--like a slice--but it's just a pointer, and doesn't keep track of the length itself, so there are no bounds checks.  

The `.ptr` field of a Zig slice is this type of pointer.  

You can do pointer arithmetic on a many-pointer.  

A many-pointer is somewhat like the pointer you get in C, when an array decays into a pointer - though, a many-pointer isn't necessarily pointing at the first element of the array; it's more about how the pointer is used, than where it points.

## How can I convert strings with sentinel ?

```zig
const std = @import("std");
const print = std.debug.print;
pub fn main() !void {
    var t1 = [4:0]u8{ 0, 0, 0, 1 };
    print("t1 type: {}\n", .{@TypeOf(t1)});
    var t2: u32 = @bitCast(u32, @as([4]u8, t1[0..4].*));
    var t3 = @ptrCast(*align(1) u32, &t1).*;
    print("t2: {d}\n", .{t2});
    print("t3: {d}\n", .{t3});
    // 00000001 00..00
    //  7 0s  1  24 0s
    print("same as t4: {d}\n", .{std.math.powi(u32, 2, 24)});
}
```

## How to explicitly ignore expression values?

Use
```zig
 _ = expression
```
See https://github.com/ziglang/zig/issues/1825