This is a collaborative effort to note quick answers to questions about programming in Zig that get asked frequently by newcomers.  
If you are about to answer a question for the third time in a row, write it here and then paste a link to cache it for the future :)  
_(maybe start by considering if it wouldn't be better to just edit an existing answer)_

## How do I parse a number from a string?
Use [`std.fmt.parseInt`](https://github.com/ziglang/zig/blob/3bf72f2b3add70ad0671c668baf251f2db93abbf/lib/std/fmt.zig#L1357) or [`std.fmt.parseFloat`](https://github.com/ziglang/zig/blob/3bf72f2b3add70ad0671c668baf251f2db93abbf/lib/std/fmt/parse_float.zig#L341).

## How do I do X with strings?
In Zig, strings are just bytes, so the stdlib namespace you want to look at is `std.mem`, it has functions for concatenation, substring replacement, comparison, etc.

The other namespace of interest to you will be `std.fmt` for formatting functions.

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


## What are the benefits of specifying the error set at the function return type?

Consider these two function declarations:

```zig
fn createFile() !void { ...

fn failFnCounter() error{Oops}!void { ...
```

This makes a library much easier to handle because you can just look in one (or a few) places for all errors the function will return.

Explicit error sets are good for API boundaries, because they ensure:
1) consumers can easily see all your errors
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

## What is `[:0]T`, `[*:0]T`, and `[N:0]T`?

They are a sentineled slice, a sentineled multipointer, and a sentineled fixed-size array, respectively.

This means that they semantically SHOULD have an extra element, after the final element, that has a specific value.

That specific value is referred to as the "sentinel" -- the `0` in `[:0]T`.

Sentinels are sometimes used to mark the end of a sequence, rather than using a count; e.g: a C-string uses a null byte to indicate where the end of the string is.

A sentinel can be any value that is of the same type as one of the elements; e.g: a C-string is a sequence of `u8`s, followed by a sentinel `u8`, that has a value of `0`.

The sentinel value resides at `x[x.len]`.

Slices that have sentinels, do NOT use the sentinel to determine the length of the slice; they are just a normal slice, with the addtional semantic that there SHOULD be an extra element, whose value is that of the of the sentinel.

## Converting from `[*:0]T` to `[]T`

You can use the Standard Library function `std.mem.sliceTo` to scan for the sentinel, and slice it at that location:
```zig
var cstr: [*:0]const u8 = "Hello\x00World";
// 'cstr' is a multipointer to immutable-byte, with a zero-terminator.

const str = std.mem.sliceTo(cstr, 0);
// 'str' is now "Hello"
```
`sliceTo` works on any sentineled slice, sentineled multipointer, or C-pointer type.

`var cstr: [:0]const u8 = "Hello\x00World";` is also legal, but the slice will have length 11, not 5.  
`std.mem.sliceTo(cstr, 0)` still correctly performs the conversion, however.

## Converting from `[]T` to `[*:0]T`

### If you don't know if the slice contains a sentinel or not (which is quite common), you'll have to duplicate it:

#### You can do this with an allocator:

```zig
const s: []const u8 = "Hello, World!";
const cstr = try allocator.dupeZ(s);
// 'cstr' is now a '[:0]u8' which is a duplicate of the original string, just with a zero-byte at the end.
// It's newly allocated, so you'll probably want to free it at some point with `allocator.free(cstr);`, depending on what allocator you use.
```
You can then use `cstr.ptr` to get a `[*:0]u8`, which can be passed to C where as C-string is expected.

#### Or with a buffer, and slicing, if you have a comptime-known maximum size:

```zig
var s: []const u8 = "Hello!";

var buf: [16:0]u8 = undefined;
if (s.len >= buf.len) @panic("not enough space");

std.mem.copy(u8, buf[0..s.len], s);
buf[s.len] = 0;

const cstr = buf[0..s.len :0]; // Asserts that buf[s.len] == 0, and gives buf[0..s.len].
// 'buf':  { 72, 101, 108, 108, 111, 33, 0, 170, 170, 170, 170, 170, 170, 170, 170, 170 }
// 'cstr': { 72, 101, 108, 108, 111, 33 }

// 'cstr' is once again a '[:0]u8`, which is a stack-based duplicate of the original string, with a zero-byte at the end.
// Since 'buf' is just an array on the stack, no freeing is required here.
```

### If you do know, then you can just use `std.mem.indexOfScalar` and slicing:
```zig

const s: []const u8 = "Hello\x00World!";
const index = std.mem.indexOfScalar(u8, s, 0) orelse @panic("no sentinel found");
const cstr = s[0..index :0]; // Asserts that s[index] == 0, and gives s[0..index].

// 's':    { 72, 101, 108, 108, 111, 0, 87, 111, 114, 108, 100, 33 }
// 'cstr': { 72, 101, 108, 108, 111 }
```

## How to explicitly ignore expression values?

Like this:
```zig
 _ = expression
```
See https://github.com/ziglang/zig/issues/1825