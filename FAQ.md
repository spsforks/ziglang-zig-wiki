## Where is the documentation for the Zig standard library?

There is no stdlib documentation yet, but [it is planned for the next release](https://github.com/ziglang/zig/issues/21). For now, the best ways to learn about the standard library are:

- Browse [the stdlib code](https://github.com/ziglang/zig/tree/master/std) to see what public functions and types are available.
- Check out [the community](https://github.com/ziglang/zig/wiki/Community) and ask questions if you need help. Newcomers are always welcome!

## Why does Zig force me to use spaces instead of tabs?

After [a very lengthy discussion](https://github.com/ziglang/zig/issues/544) about tabs and spaces, it was decided that only spaces would be allowed.

> The biggest reason to enforce an indentation and line endings is that it eliminates energy spent on debating what the standard should be, since the standard is enforced by the compiler.

The issue of [other whitespace characters has been discussed too](https://github.com/ziglang/zig/issues/663), and similar decisions were made. Zig aims to offer only one way to do things whenever possible. This makes the cognitive load lower for programmers and keeps the compiler code base simpler and easier to understand.

Note that [it is planned to have `zig fmt` allow for tabs](https://github.com/ziglang/zig/issues/2819) (as well as a few other illegal, but unambiguous, whitespace characters) and automatically convert them.

## How do I make `zig fmt` skip a range of source lines?

`zig fmt` will parse comments for special directives. In this example all code between `// zig fmt: off` and `// zig fmt: on` will be excluded from formatting:

```zig
// zig fmt: off
const matrix = Matrix{1.0, 0.0, 0.0, 0.0,
                      0.0, 1.0, 0.0, 0.0,
                      0.0, 0.0, 1.0, 0.0,
                      0.0, 0.0, 0.0, 1.0};
// zig fmt: on
```