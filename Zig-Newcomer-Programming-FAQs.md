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

The "many" part of this comes from the fact that its effectively a pointer-to-an-array (`*[N]T`), but where the count isn't part of the type; instead, it's of unspecified size.  
This is usually because the size is determined some other way, such as null-termination, or a separate integer.

"A slice without a length;" it can be indexed into or sliced--like a slice--but it's just a pointer, and doesn't keep track of the length itself, so there are no bounds checks.  

The `.ptr` field of a Zig slice is this type of pointer.  

You can do pointer arithmetic on a many-pointer.  

A many-pointer is somewhat like the pointer you get in C, when an array decays into a pointer - though, a many-pointer isn't necessarily pointing at the first element of the array; it's more about how the pointer is used, than where it points.

## What is a `[N]T`?

It's an array, consisting of `N` x `T`s in a row in memory.  
An array is a value type; it _doesn't_ point to memory somewhere else; it's just a block of `T`s.

The length of the array is known at compile-time.

You may slice an array to get a slice that refers to the memory of the array.

The elements are always `T` - they cannot be `const T` - unlike a slice.  
This means that unless an array variable is declared as `const`---or if you take the address of a temporary---the elements are mutable.

A pointer to an array will coerce to a slice.

Indexing an array is bounds-checked, and will panic if it is out of bounds, or produces a compile error if the index is known at compile-time.

## What is a `[_]T`?

`[_]T` isn't actually a type, but simply a way to indicate to Zig that you want an `[N]T`, where the the `N` is inferred.

This can currently only be used with array literals; e.g: `var array = [_]i32{1, 4, 9};`.

## What is a `[]T`?

It's a slice, which is a structure consisting of two fields: a many-pointer (`ptr: [*]T`), and a length (`len: usize`).  
It points to a block of memory that's somewhere else.

In essence, it's like a pointer-to-an-array (`*[N]T`) but instead of the length being part of type, and known at compile-time, it's stored as part of the slice structure itself, as the `len` field.

The length is generally only known at runtime, unless a slice variable is `comptime var`.

You can slice a slice to get another slice, which only refers to part of the original slice.  
If you slice with compile-time known indices, you get a pointer to an array instead.

A slice can be of `T`s, or `const T`s. i.e: `[]u8`, `[]const u8`.  
You cannot mutate the elements of the slice if they are `const` in this way, similarly to how you cannot mutate a `T` through a `*const T`.

Indexing a slice is bounds-checked, and will panic if it is out of bounds.

This type is quite common to see in Zig code as a function argument, since a slice can be obtained from any block of memory, be it on the stack, dynamically allocated, memory-mapped, etc - so it allows a function to operate on it without having to know or care where precisely it's located, which is good for writing reusable code.

## How can I convert between a string with sentinel, and one without?

A "sentinel" is an extra element in the array or slice, at index `items[items.len]`, which typically marks the end of a string.

Multipointers (`[*]T`), slices (`[]T`), and fixed-size arrays (`[N]T`) can all have sentinels.

It's expressed in a type with `:S` syntax.
e.g: `[*:0]u8`, `[:0]u8`, `[N:0]u8`.

### Removing the sentinel from the equation (e.g: cstring -> string)
You can use the Standard Library function `std.mem.sliceTo` to scan the string for the sentinel, and slice it at that location:
```zig
var cstr: [*:0]const u8 = "Hello\x00World";
// 'cstr' is a multipointer to immutable-byte, with a zero-terminator.

const str = std.mem.sliceTo(cstr, 0);
// 'str' is now "Hello"
```
`sliceTo` works on any slice, multipointer, or C-pointer type, provided the type has a sentinel.  
Changing `cstr` to be a slice will still work as expected: `var cstr: [:0]const u8 = "Hello\x00World";`

### Adding a sentinel
Sometimes you've got a string, and need to pass it to C as a C-string; i.e: it needs null-termination.  
If you have no control over the memory, you'll need to duplicate it:
```zig
const s: []const u8 = "Hello, World!";
const cstr = try allocator.dupeZ(s);
// 'cstr' is now a '[:0]u8' which is a duplicate of the original string, just with a zero-byte at the end.
// It's newly allocated, so you'll probably want to free it at some point with `allocator.free(cstr);`, depending on what allocator you use.
```

If you have a buffer, and want to use it to temporarily store a C-string, you can use `sliceTo` again to determine its length from the null-terminator:
```zig
var buf: [8:0]u8 = std.mem.zeroes([8:0]u8);
std.mem.copy(u8, buf[0..], "Hello!");

//
// now we want to get a null-terminated string from this.
//

const sentineled = buf[0..buf.len :0];
// this yields a null-terminated value; either a slice or pointer-to-array, depending on whether the indices are comptime-known or not.
//  we need it to be null-terminated for calling `sliceTo`.
// we're slicing buf[0..8] (all 8 elements), and checking that the 9th element is zero: `buf[8] == 0`.
//  `buf[7]` (the 8th element!) will be the last element of this slice.
//  the 9th element of the array is added by the '0' in the array's type `[8:0]u8`; this is actually 8+1 u8s.

const cstr = std.mem.sliceTo(sentineled, 0);
// 'cstr' is now "Hello!"
```

## How to explicitly ignore expression values?

Use
```zig
 _ = expression
```
See https://github.com/ziglang/zig/issues/1825