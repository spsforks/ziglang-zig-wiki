It wouldn't be more prettier to add something like backward iteration or at least backward enumeration. Why when I want to iterate backward I have to do extra code which affect on readability.

We could have something like this for iteration:
```zig
var arr: [4]u32 = .{ 0, 4, 5, 1 };
for (..arr) |a| {
    std.debug.print("{}", .{a});
}
```

```
output: 0451
```

or at least something like this for enumeration:
```zig
var arr: [4]u32 = .{ 0, 4, 5, 1 };
for (3..0) |i| { // with enumeration logic of current zig it should be 3..-1, but ...
    std.debug.print("{}", .{arr[i]});
}
```

I don't think it would affect on simplicity of zig.
