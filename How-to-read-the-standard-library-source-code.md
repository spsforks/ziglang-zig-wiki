# How to read the standard library source code

## Where is the stdlib code located?
You can find it under [`/lib/std/`](https://github.com/ziglang/zig/tree/master/lib/std).

## How is the stdlib structured?
When you import "std" in a Zig source file, you are importing `/lib/std/std.zig`.
In that file you can see where each exposed symbol comes from.
 
Some parts of the stdlib are simple and are implemented in a single file, like `std.ascii`, which is entirely implemented in `/lib/std/ascii.zig`. Some other parts of the standard library are beefier and so have their own dedicated subdirectory, like `std.math`.
Those beefier "submodules" always have a regular structure: 
- A subdirectory that contains most of the implementation
- A file inside `/lib/std/` that exposes the public parts implemented in the relative subdirectory

In the case of `std.math`, for example, `/lib/std/math/ceil.zig` contains the implementation of ceil that then `/lib/std/math.zig` re-exports like this:

```zig
pub const ceil = @import("math/ceil.zig").ceil; 
```

With this knowledge it should be easy for you to track down the implementation code of every symbol defined in the standard library.

One final trick: search for `pub fn` and `pub const` to quickly skim over the public API of an implementation.

## Is there any other material on the topic?
- [Reading Zig's Standard Library (video)](https://www.youtube.com/watch?v=NQgju_2mX-8)