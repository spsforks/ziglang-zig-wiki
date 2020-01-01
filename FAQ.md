## Where is the documentation for the Zig standard library?

Here it is: [Standard Library Documentation](https://ziglang.org/documentation/master/std/)

However, please note the [remaining issues with generated documentation](https://github.com/ziglang/zig/issues/21#issuecomment-539735292).

You can also:

- Browse [the stdlib code](https://github.com/ziglang/zig/tree/master/lib/std) to see what public functions and types are available.
- Check out [the community](https://github.com/ziglang/zig/wiki/Community) and ask questions if you need help. Newcomers are always welcome!

## Why does Zig force me to use spaces instead of tabs?

Because the tooling is not yet stable.

`zig fmt` accepts and converts tabs to spaces, as well as many other transformations of non-canonical to canonical style.

The Zig language accepts hard tabs. The self-hosted compiler implements the Zig language correctly; accepting hard tabs. However, the self-hosted compiler is not yet complete, and what people are using in reality is the stage1 compiler, which does not accept hard tabs.

Hard tabs are not accepted by stage1 because:
 * It doesn't need to. All of the self-hosted compiler source is formatted with `zig fmt` and so there are no hard tabs. The complexity of dealing with hard tabs need not be present in stage1.
 * `zig fmt` is not fully stable yet; use of `zig fmt` is not yet ubiquitous. If stage1 accepted hard tabs, then in practice, there would be accidental mixing of tabs and spaces.

If you feel the need to spend any more of your precious hours left on this Earth thinking about tabs and spaces, see [The Hard Tabs Issue](https://github.com/ziglang/zig/issues/544).

## Why does `zig fmt` have no configuration options?

The biggest reason to enforce an indentation and line endings is that it eliminates energy spent on debating what the standard should be, since the standard is enforced by the compiler.

The issue of [other whitespace characters has been discussed too](https://github.com/ziglang/zig/issues/663), and similar decisions were made. Zig aims to offer only one way to do things whenever possible. This makes the cognitive load lower for programmers and keeps the compiler code base simpler and easier to understand.

In the words of the Go community,

> `gofmt` is nobody's favorite, yet `gofmt` is everybody's favorite.

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

The build cache can be found in the following locations unless overridden with command-line options:

TYPE | OS | DIRECTORY
:-: | :-: | ---
global | linux | $XPG_CACHE_HOME/zig or $HOME/.cache/zig<br>(as of master branch 0.5.0+280)
|| macOS | $HOME/Library/Application Support/zig
|| Windows | %LOCALAPPDATA%\zig
local | all | $PWD/zig-cache

## Why is switching on `[]u8` (strings) not supported?

In summary, Jimmi made a good attempt at implementing a `StringSwitch` in `comptime` and concluded that good old chained `if` statements were fastest.

For details see [match.zig](https://github.com/Hejsil/fun-with-zig/blob/master/bench/match.zig) .

## Are there any good examples of advanced internals development with Zig (specifically stage1 bug fixes)?

Zig stage1 compiler is currently implemented in c++ and will probably remain that way for some time. If you want to see some good examples of fixing a bug in the stage1 compiler:

- case where zig IR → LLVM-IR is bugged: [issue #2791](https://github.com/ziglang/zig/issues/2791)
- [how to create a small zig file for viewing IR](https://gist.github.com/andrewrk/5684434a2a8d4cbb08bb0d855c4f2ada)

## I would like to use `--verbose-ir` but it is really loud. How can I focus the compiler?

1. The first step is to reduce your code to isolate the issue as much as possible.
2. The second step is to setup your code to not require an executable with `main()` or similar. Instead we define an `export` function to force the compiler into thinking the function must be compiled. Note this means you are probably going to have to use `zig build-obj` instead of `zig build-exe` or `zig run`.
3. The third step is to define a skeleton panic handler to override the more functional default.

Here is a reduction boiler plate. Note little tricks like using easily seen variable names or literal values can help. For example, `a` is a difficult variable to grep for. And the value `99` is much easier to find than `0`:

```zig
export fn entry() void {
    var hello: usize = 99;
}

pub fn panic(msg: []const u8, error_return_trace: ?*@import("builtin").StackTrace) noreturn {
    while (true) {}
}
```

and search for `fn entry` (you'll see it twice because zig first produces "IR0" from source then analyzes IR0 and produces IR). Either or both may be of interest depending on the issue at hand:

```
zig build-obj reduction.zig --verbose-ir |& less
```