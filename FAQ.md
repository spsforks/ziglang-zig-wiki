## Where is the documentation for the Zig standard library?

There is no stdlib documentation yet, but [it is planned for the next release](https://github.com/ziglang/zig/issues/21). For now, the best ways to learn about the standard library are:

- Browse [the stdlib code](https://github.com/ziglang/zig/tree/master/lib/std) to see what public functions and types are available.
- Check out [the community](https://github.com/ziglang/zig/wiki/Community) and ask questions if you need help. Newcomers are always welcome!

## Why does Zig force me to use spaces instead of tabs?

After [a very lengthy discussion](https://github.com/ziglang/zig/issues/544) about tabs and spaces, it was decided that only spaces would be allowed.

> The biggest reason to enforce an indentation and line endings is that it eliminates energy spent on debating what the standard should be, since the standard is enforced by the compiler.

The issue of [other whitespace characters has been discussed too](https://github.com/ziglang/zig/issues/663), and similar decisions were made. Zig aims to offer only one way to do things whenever possible. This makes the cognitive load lower for programmers and keeps the compiler code base simpler and easier to understand.

Note that [as of 2019-07-05](https://github.com/ziglang/zig/commit/4f43a4b30f8a6dad7a9a35ccf1cef89b6d239997), when running `zig fmt`, tabs and carriage returns [will be accepted and converted automatically](https://github.com/ziglang/zig/issues/2819).

## Why are some `zig` command options prefixed with `-` and others with `--` ?

They were copied verbatim from other projects rather than trying to be self-consistent. The Command Line Interface is not finalized. We can make sure it is consistent and intuitive in an organization pass before releasing 1.0.

## How do I make `zig fmt` skip a range of source lines?

`zig fmt` will parse comments for special directives.

In this example all code between `// zig fmt: off` and `// zig fmt: on` will be excluded from formatting:

```zig
// zig fmt: off
const matrix = Matrix{1.0, 0.0, 0.0, 0.0,
                      0.0, 1.0, 0.0, 0.0,
                      0.0, 0.0, 1.0, 0.0,
                      0.0, 0.0, 0.0, 1.0};
// zig fmt: on
```

## Explain this error: `Unable to create builtin.zig: access denied`

When building or compiling with Zig a build-cache is used.
This particular error indicates filesystem security has prevented access
to a directory or file used in the *global* build-cache.
Fixing permissions should solve the issue.

note: It is safe to manually remove cache directories when no zig compiler process is active.

As of Zig 0.4.0 the build cache can be found in the following locations unless overridden with command-line options:

TYPE | OS | DIRECTORY
:-: | :-: | ---
global | linux | $HOME/.local/share/zig
|| macOS | $HOME/Library/Application Support/zig
|| Windows | %LOCALAPPDATA%\zig
local | all | $PWD/zig-cache

## Why is switching on `[]u8` (strings) not supported?

In summary, Jimmi made a good attempt at implementing a `StringSwitch` in `comptime` and concluded that good old chained `if` statements were fastest.

For details see [match.zig](https://github.com/Hejsil/fun-with-zig/blob/master/bench/match.zig) .

## Are there any good examples of advanced internals development with Zig (specifically stage1 bug fixes)?

Zig stage1 compiler is currently implemented in c++ and will probably remain that way for some time. If you want to see some good examples of fixing a bug in the stage1 compiler:

- case where zig IR → LLVM-IR is bugged: [issue #2791](https://github.com/ziglang/zig/issues/2791)