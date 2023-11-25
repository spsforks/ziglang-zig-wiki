# Bug Reports
Bug reports are welcome but please keep in mind the following:

At the moment Autodoc is not using semantic analysis information. 
Because of that, Autodoc is expected to give up analysis as soon as it encounters
any non-trivial comptime expression. 

Bug reports about missing information resulting from comptime logic will be closed.
Some exceptions apply but if you're not sure about a potential bug report, consider
asking for advice in the #autodocs channel of the big Zig Discord server.
 
# Intro
Hello friend, welcome to the fantastic world of Autodoc.
Before we begin with our adventure you need to be aware that contributing to Autodoc means
- Writing JS code.
- Deal with not-yet-fully-designed systems.
- [Be (willing to become) familiar with ZIG](https://mitchellh.com/zig).

If you're fine with that, then welcome aboard!

## The Grand Autodoc Plan
Autodoc will show:
1. Information about types and decls in the code
2. Handwritten guides (from markdown files)
3. Syntax-highlighted source listings

Right now we're working on [1]. 

[1] is implemented in two stages.
The first stage reads ZIR data and outputs the main corpus of definitions that the user will see.
The code might contain comptime expressions that require comptime analysis to fully resolve.
This is where the second phase comes into play: we [*] keep track of unsolved expressions and wait
for Semantic analysis to happen so that we can get that information aswell.

*[\*] we plan to, this is not implemented yet.*


The benefit of relying on ZIR is that we can care about details that Sema doesn't care about, such as preserving indirection (eg `const Foo = u8;` vs `const Foo = X; const X = u8;`) as it might have semantic meaning from the perspective of the documentation.

## Where is Autodoc?

Autodoc is currently made up of 4 files:
- `src/Autodoc.zig`
- `src/autodoc/render_source.zig`
- `lib/docs/main.js`
- `lib/docs/index.html`

`Autodoc.zig` is invoked by `Compilation.zig` between Astgen and Sema, and its output is going to be a JSON payload consumed by `main.js` and `index.html`, which contain the frontend logic.

The documentation is **not** pre-rendered for performance reasons. The JS code will instead navigate the data structure and use that information to render all individual possible "views" of it upon request.

## Calling Autodoc

1. (optional) Create a release build of stage1. Release helps with the rebuild-speed of stage2.
2. Build stage2 (and rebuild it every time you change `Autodoc.zig`)
   - `./zig build -p zig-out -Dno-lib -Dlog` 
3. Build a Zig file with stage2 and enable docs (here `zig2` is a symlink to `build/zig-out/bin/zig`)
   - `zig2 build-obj -femit-docs foo.zig`
4. Open the newly-generated `docs/index.html`.

NOTE: Now autodoc is also available on stage1 builds, but for faster rebuilds it is still recommended to build stage2 after every change (and use that instead of stage1 to run autodoc).

## How to contribute

Create a tiny Zig script with some decls in it and run it through `-femit-docs` and see if the output is missing some information. If you need more inspiration, run it on an existing project (and then produce a reduced version). Discuss with @kristoff-it and other contributors how to improve the feature. Any semantic improvement over status-quo is good, even if we don't know yet what the perfected version should look like (ie let's first get the data, then decide how to best render it).

Some information is missing because the frontend doesn't know how to display it, but there's also a sizeable chunk of the language that is not yet supported by the backend. In this second case you will see that calling `-femit-docs` will produce a bunch of logs that will hint at the not-yet-supported constructs.

Example:

```
$ zig2 build-obj -femit-docs src/okredis.zig
basename: docs
Context [client.zig] % 44
TODO: implement `struct_init_anon` for walkInstruction


Context [types/reply.zig] % 18
TODO: implement `error_value` for walkInstruction
```

These messages tell you that, for example: 
- `client.zig` in ZIR position 44 has a `struct_init_anon` instruction that Autodoc is not handling correctly inside the body of the `walkInstruction` function.

Since producing the JSON output requires intimate knowledge of ZIR, one handy trick is to call:

`zig2 ast-check -t client.zig`

This will show a textual representation of ZIR, helping you understand the context of the unhandled instruction.
This goes hand-in-hand with the messages printed by `Autodoc.zig`!

## How does Autodoc.zig work internally?

The main entry point is `generateZirData()` which will start from the first instruction of the root file and recursively analyze the entire project. This mostly means calling into `walkInstruction()`.

The main non-obvious concept in Autodoc is `WalkResult`. It is a union of all the possible outcomes that can come from analyzing an expression. Since we want to preserve indirection (as mentioned above), we have some cases that are specific to Autodoc, the most important of which is `refPath`.

Additionally, since some expressions will have to remain unresolved (because of comptime), we also have the concept of `ComptimeExpr`. 

The main way indirection (`refPath`) and `ComptimeExpr` complicate our job is by forcing us, when rendering, to account for all the cases where a "piece" of an expression is referred to indirectly or is unknown. As an example:

```zig
const MyArrType = [xxx(5)]Foo.Bar;

const Foo = struct {
   const Bar = u8;
};
```

In this case we know that `MyArrType` is an Array type, and we also know that its child type is `u8`, but we don't know the length, as that would require evaluating `xxx(5)`, which we're not doing at this stage.

This means that for us, an array type has to be defined as follows:

```zig
const ArrayType = struct {
   len: WalkResult, // WalkResult { call: .{ args: []WalkResult{ .{int: 5} }, callee: WalkResult {declRef: "xxx"} } }, 
   child: WalkResult, // WalkResult { refPath: []WalkResult{ .{declRef: "Foo"}, .{declRef: "Bar"} } };
};
// NOTE: the real implementation of WalkResult is slightly different
```

## Running Autodoc on the stdlib
`zig2 build-obj -femit-docs std.zig`, simple as that. Make sure to open the correct `index.html` after that.

## Good starting point?
A good starting point would be to add all the missing qualifiers from types, decls and function arguments. Stuff like `alignment`, for example. Right now the data model is missing all this "secondary" qualifiers, etc.




