_Written against commit [0.6.0+65d8271](https://github.com/ziglang/zig/commit/65d827183bd521cd402826382541d8b16d7718bb) from 2020-05-25._

## Summary
Zig build scripts are written in Zig, and are themselves entirely ordinary Zig programs utilizing [`std.build.Builder`](https://github.com/ziglang/zig/blob/master/lib/std/build.zig) to express all imaginable ways to turn your source into executables, libraries, or other things. We'll explore how that works and look at interesting options the build system offers that you might not know, starting out with a brief summaries on common problems.

## How do I do X?
_In case you find out about something you think might be useful for posterity, feel free to add it!_  
Relevant sources include [std/build.zig](https://github.com/ziglang/zig/blob/master/lib/std/build.zig) for `Builder`, `LibExeObjStep` and some other steps, [std/build/](https://github.com/ziglang/zig/tree/master/lib/std/build) for various other steps, [std/special/init-exe/build.zig](https://github.com/ziglang/zig/blob/master/lib/std/special/init-exe/build.zig) for the default application build.zig,  [std/special/init-lib/build.zig](https://github.com/ziglang/zig/blob/master/lib/std/special/init-exe/build.zig) for the default library build.zig and [std/special/build_runner.zig](https://github.com/ziglang/zig/blob/master/lib/std/special/build_runner.zig) for the file executed when you run `zig build`.

#### Use a Zig library
Use `LibExeObjStep.addPackage()` with a `Pkg{ .name = "library", .path = "/path/to/the/library"}`. `LibExeObjStep.addPackagePath()` *should* work too; then use `const library = @import("library");` in your root source file.

#### Use a native (C) library
Use `LibExeObjStep.linkSystemLibrary()` with your library's name and `@cInclude()` in your source code.

#### Use build-time custom command line flags (`-Dsomething`)
Use `LibExeObjStep.addBuildOption()` to add a value to the `build_options` package. To get this value from the building user, use `Builder.option()`. Supported types for `option` are Strings and Enums (`-Dname=value` style), Booleans (`-Dname`, `-Dname=true`, `-Dname=false` style) and list of strings (`-Dname=value -Dname=value2` style). Use from your source code like `const should_do_thing = @import("build_options").do_thing;`

#### Run commands as build steps
Use `Builder.addSystemCommand()` to get a step that runs your command, then create a top level step using `b.step()`, then make the top level step depend on your run step using `top.dependOn(&run.step)`

## `build.zig` and how we get to it
Let's take a look at the default `build.zig` for applications over in [lib/std/special/init-exe/build.zig](https://github.com/ziglang/zig/blob/master/lib/std/special/init-exe/build.zig):
```zig
const Builder = @import("std").build.Builder;

pub fn build(b: *Builder) void {
    // modify b and add our desired settings
}
```
At first, we import the aforementioned `std.build.Builder`. This is a struct representing a pending build and all of its associated steps and their respective settings. Then we create a function that takes one of these Builders and adds our build logic into it.  
Then, after we return from this function, *magic happens* and our instructions are executed. Well, not exactly magic; all invoking `zig build` does under the hood is building and running [lib/std/special/build_runner.zig](https://github.com/ziglang/zig/blob/master/lib/std/special/build_runner.zig), which is pretty much just a normal Zig application with `pub fn main()` and all the things you might already know from your actual project.   
This `build_runner` imports your project's `build.zig` (it does this with a magic `@import("@build")`, which isn't standard syntax, but immediately after that it gets normal again and stays that way) and somewhere in its belly invokes your `pub fn build(b: *Builder)` on a `Builder` it created earlier. The very last thing it does is hand over to this `Builder` you got to modify using `make()`.

## How `Builder` works
Looking at [`std.build.Builder`](https://github.com/ziglang/zig/blob/master/lib/std/build.zig) can be overwhelming, after all the file comes in at 2.5k lines at time of writing. Its main functions (in the writer's opinion, that is) however are these three:
1. Coordinate and execute `Step`s that describe different stages of a build
2. Provide default target and release mode for `Step`s
3. Provide `build_options`
We'll walk through all of these now.

### What is a `Step`?
A [`std.build.Step`](https://github.com/ziglang/zig/blob/master/lib/std/build.zig) (defined close to the bottom of the file) is a struct with two noteworthy properties: a `makeFn` that does the actual work which implementing this step entails and `dependencies`, an `ArrayList` of different `Step`s that must be executed before this one (though that isn't handled by `Step` itself).  
The base `Step` struct isn't actually that interesting, you'll mostly use structs that wrap a bare `Step` like `BuildExeObjStep` (this is the big one that actually does all of the compiling work), `LogStep` (very simple step that writes something to stderr) or `RunStep` (which runs a system command) — actually in fact you probably won't construct any of these structs yourself and just use one of the manifold convenience methods on `Builder` to create such a step and add it to the builder, like `builder.addTranslateC(std.build.FileSource)`. To get a quick overview of them, Ctrl-F for `pub fn add` while in the source file.

#### `LibExeObjStep`
This step is the main one you'll be using most likely. It is capable of invoking the zig compiler on your sources and turning them into executables or shared objects/DLLs. After you've obtained a `LibExeObjStep` with one of `Builder`s `addX` methods, you can adjust its myriad settings to your liking and finally call `install()` to create a build artifact in `./zig-cache/bin` (this path is also adjustable using `setOutputDir`).  
You can also use a `LibExeObjStep` to run your tests as done in the [default build.zig for libraries](https://github.com/ziglang/zig/blob/master/lib/std/special/init-lib/build.zig).  

#### Other Steps
TODO.

## Targets with `build.zig`
The default `build.zig` template exposes the full power of Zig's cross-compiling to the building user — maybe that's not strictly necessary or even just incorrect if your project can't be meaningfully cross-compiled anyway. You can use `LibExeObjStep.setTarget(std.CrossTarget)` to precisely define what your project should be built for; the easiest way is calling it with [`std.CrossTarget`](https://github.com/ziglang/zig/blob/master/lib/std/zig/cross_target.zig)`.parse(std.CrossTarget.ParseOptions)` to get a interface reminiscent of the `-target` CLI option. The `ParseOptions` struct is fairly well documented in the source.

## `build_options`
To provide compile-time configuration, the build system can create a package called `build_options` to communicate values from `build.zig` to your project's source code. This package's declarations are populated by calling `LibExeObjStep.addBuildOption(type, name, value)` in your build.zig.  
You can even provide user input for these in form of `-Dname=value` flags. For this, you can get the value a user provided (or `null` if they didn't, so use `orelse` on anything you get from this) using `Builder.option(type, name, description)`.