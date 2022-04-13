## Where can I ask questions?

This wiki page lists Zig community gathering places where you can ask questions.

https://github.com/ziglang/zig/wiki/Community

## Where is the documentation for the Zig standard library?

Here it is: [Standard Library Documentation](https://ziglang.org/documentation/master/std/)

However, please note the [remaining issues with generated documentation](https://github.com/ziglang/zig/issues/21#issuecomment-539735292).

You can also:

- Browse [the stdlib code](https://github.com/ziglang/zig/tree/master/lib/std) to see what public functions and types are available.
- Check out [the community](https://github.com/ziglang/zig/wiki/Community) and ask questions if you need help. Newcomers are always welcome!

## Why does Zig force me to use spaces instead of tabs?

Because no human and no contemporary code editor is capable of handling tabs correctly. Humans tend to mix tabs and spaces on accident, and editors don't have a way to "indent with tabs, align with spaces" without pressing the space bar many times, leading programmers to use tabs for alignment as well as indentation.

Tabs would be better than spaces for indentation because they take up fewer bytes. But in practice, what ends up happening is incorrectly mixed tabs and spaces. In order to simplify everything, tabs are not allowed. Spaces are necessary; we can't ban spaces. But tabs are not strictly needed, so the null hypothesis is to not have them.

Maybe someday, we'll switch to tabs for indentation, spaces for alignment and make it a compile error if they are incorrectly mixed. But if we did that today, writing Zig code would be too hard. For now your options are to configure your editor to insert spaces when you press the tab key, or configure your editor run `zig fmt` on save (recommended).

Currently, the stage1 parser rejects tabs and the stage2 parser accepts them. What will make it into the final language specification? It isn't decided yet and it doesn't really matter. Just run `zig fmt` on save.

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

TYPE | OS | DIRECTORY | NOTE
:-: | :-: | --- | ---
global | all | $XDG_CACHE_HOME/zig | if env is set
||| $HOME/.cache/zig
|| Windows | %LOCALAPPDATA%\zig
local | all | $PWD/zig-cache

## Why is switching on `[]u8` (strings) not supported?

In summary, Jimmi made a good attempt at implementing a `StringSwitch` in `comptime` and concluded that good old chained `if` statements were fastest.

For details see [match.zig](https://github.com/Hejsil/fun-with-zig/blob/17f524b0ea394cbf4e49b4ac81dfbd39dcb92aa4/bench/match.zig) . Note that switching on [variable identifiers as const strings](https://ziglang.org/documentation/master/#toc-Identifiers) is easy with the `@""` syntax.

## Are there any good examples of advanced internals development with Zig (specifically stage1 bug fixes)?

Zig stage1 compiler is currently implemented in c++ and will probably remain that way for some time. If you want to see some good examples of fixing a bug in the stage1 compiler:

- case where zig IR → LLVM-IR is bugged: [issue #2791](https://github.com/ziglang/zig/issues/2791)
- [example of how to update zig_clang.cpp when assertions fail](https://lists.sr.ht/~andrewrk/ziglang/%3Cfdb6582a-3f17-703c-4752-2c1af13e09be%40ziglang.org%3E)

## I would like to use `--verbose-air` but it is really loud. How can I focus the compiler?

1. The first step is to reduce your code to isolate the issue as much as possible.
2. The second step is to setup your code to not require an executable with `main()` or similar. Instead we define an `export` function to force the compiler into thinking the function must be compiled. Note this means you are probably going to have to use `zig build-obj` instead of `zig build-exe` or `zig run`.
3. The third step is to define a skeleton panic handler to override the more functional default.

Here is a reduction boiler plate. Note little tricks like using easily seen variable names or literal values can help. For example, `a` is a difficult variable to grep for. And the value `99` is much easier to find than `0`:

```zig
export fn entry() void {
    var hello: usize = 99;
}

pub fn panic(msg: []const u8, error_return_trace: ?*@import("std").builtin.StackTrace) noreturn {
    @breakpoint();
    unreachable;
}
```

and search for `fn entry` (you'll see it twice because zig first produces "IR0" from source then analyzes IR0 and produces IR). Either or both may be of interest depending on the issue at hand:

```
zig build-obj reduction.zig --verbose-air |& less
```

## Why was varargs replaced with tuples?

see https://github.com/ziglang/zig/issues/208#issuecomment-393777148

## Why do I get `illegal instruction` when using with `zig cc` to build C code?

When compiling without `-O2` or `-O3`, Zig infers [Debug Mode](https://ziglang.org/documentation/master/#Debug). Zig passes `-fsanitize=undefined -fsanitize-trap=undefined` to Clang in this mode. This causes Undefined Behavior to cause an Illegal Instruction. You can then run the code in a debugger and figure out why UB is being invoked.

From the [UBSAN docs](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html):

> UndefinedBehaviorSanitizer is not expected to produce false positives. If you see one, look again; most likely it is a true positive!

However you can suppress UBSAN. Here is how to affect the build mode that Zig selects for C code:

 * `-O2` or `-O3`: ReleaseFast
 * `-O2` or `-O3` and `-fsanitize=undefined`: ReleaseSafe
 * `-Os`: ReleaseSmall
 * `-Og` or no optimization flags: Debug

You can also pass `-fno-sanitize=undefined`.

## How to add command-line args to `zig build run` sub-command?

#### usage
```sh
zig build run -- one two three
```

#### edit `build.zig`
```zig
const run_cmd = exe.run();
if (b.args) |args| run_cmd.addArgs(args);
```

## How to tie a `zig build run` sub-command to custom `-D` style options

#### usage
```sh
zig build run -Dhello=false
zig build run -Dhello=true
```

#### edit `build.zig`
```zig
exe.addBuildOption(bool, "hello", hello);
```

#### `src/main.zig`
```zig
const std = @import("std");

pub fn main() anyerror!void {
    const options = @import("build_options");
    if (options.hello) {
        std.debug.warn("Hello.\n", .{});
    } else {
        std.debug.warn("All your base are belong to us.\n", .{});
    }
}
```

## How do I use packages?

First, some basics:
- Zig packages are just Zig source trees, requiring a root file.
- Packages are imported with `@import("package-name")` without the `.zig` extension.
- Packages have the same visibility rules as other source files, they just don't need to be in a relative directory to your own source tree.
- Each package has their own dependencies, so packages might use a same-named package that is backed by different source files.

This example shows how to add two packages "first" and "second", where "second" depends on a package "first", which is different:

```
.
├── first      /path/to/one/first.zig
└── second     /path/to/second.zig
    └── first  /path/to/a/different/first.zig
```

### `zig build-exe` and other direct commands:

```sh
zig build-exe \
    --pkg-begin \
        first \
        /path/to/one/first.zig \
    --pkg-end \
    --pkg-begin \
    second \
    /path/to/second.zig \
        --pkg-begin \
          first \
          /path/to/a/different/first.zig \
        --pkg-end \
    --pkg-end
```

### `zig build`
```zig
const exe = b.addExecutable(...);

// simple package
exe.addPackage(std.build.Pkg {
    .name = "first",
    .path = "/path/to/one/first.zig",
});

// with deps
exe.addPackage(std.build.Pkg {
    .name = "second",
    .path = "/path/to/second.zig",
    .dependencies = &[_]std.build.Pkg {
        std.build.Pkg {
            .name = "first",
            .path = "/path/to/a/different/first.zig",
        },
    }
});
```

## Why am I seeing hard errors that a framework could not be found when cross-compiling to macOS from a different host OS?

Zig, and `zig ld` in particular, has the ability to link against frameworks when cross-compiling, however, a copy of the Darwin's sysroot (also called the SDK) is not bundled with Zig's installation. Therefore, it is necessary to provide the sysroot yourself, and provide Zig with valid paths to the sysroot and any search directories that may be required.

A typical invocation when linking against `Foundation` framework may look as follows,

```
$ zig cc -target aarch64-macos --sysroot=/home/kubkon/macos-SDK -I/usr/include -L/usr/lib -F/System/Library/Frameworks -framework Foundation -o hello hello.c
```