**The meetings happen weekly, every Thursday at 22.00 CET / 21.00 UTC / 13.00 PST**

Edit this wiki to get your agenda item for next week added.

## 2021-03-25
### Agenda
1. @andrewrk:
   - zir-memory-layout branch update
   - a plan for inline while, comptime code, etc in stage2

## 2021-03-18
### Agenda
1. @joachimschmidt557:
    - Extract register allocation to separate file
    - Track all registers, not just callee preserved
    - Slightly modify register spilling
2. @andrewrk:
    - Pitching an idea for handling archive files in the cache system/ incremental compilation
    - Status update on zir-memory-layout branch
    - The "start2.zig" plan
3. @kubkon
    - Pitching an idea for handling PIE targets in a uniform manner - see [#8281](https://github.com/ziglang/zig/pull/8281) for changes to GOT handling on macOS
    - Status update on `zld` - see [#8282](https://github.com/ziglang/zig/pull/8282)
4. @dimenus
    - new contributor trying to grok the stage 2 compiler pipeline
    - working on TODOs for inline asm

## 2021-03-11
### Agenda
1. @joachimschmidt557: CPU feature detection on ARM revisited (see https://github.com/ziglang/zig/pull/8206)
2. @andrewrk: progress on ZIR memory layout reworking. LLVM12 status update.

## 2021-03-04
### Agenda
1. @andrewrk:
   - Demo the new target CPU features maintenance strategy
   - LLVM 12 status update
   - Looking for contributors for native CPU feature detection for ARM
   - Some guidance on lazy value system / error sets / strategizing what areas of stage2 to work on so we don't have to redo work later
2. @joachimschmidt557:
   - wrapping arithmetic vs normal arithmetic in TZIR
   - showcase of different ways we could implement CPU feature detection for ARM
3. @kubkon:
   - Yet-another-demo of zld -- zld now passes a vast majority of Zig tests both on aarch64 and x86_64!
   - Enabling logs in stage1 for the linker stage -- as a quick workaround while working out fixes to reloc fixups, I've reverted to using log.warn in place of log.debug to gather some context for the potentially failing test cases; however, a more permanent solution would be to print logs with some runtime command/setting. So the question really is, do we have something similar to how we manage logs in stage2 in stage1?

## 2021-02-25
### Agenda
1. @andrewrk: proposal to improve test harness to support loading test cases at runtime
2. @andrewrk: merge of ast memory layout
3. @kubkon: zld upstreaming & how to handle caching logic as it applies to linking

## 2021-02-18
### Agenda
1. Loris: alternate meeting venues?
2. @andrewrk: progress report on ast memory layout
3. @andrewrk: I have to drop everything and work on llvm12 branch
4. @kubkon: https://github.com/ziglang/zig/pull/8035 -- for context, I need this to be able to integrate `zld` in Zig
5. @kubkon: `zld` progress -- the story thus far...

## 2021-02-04
### Agenda
1. @andrewrk: progress on ast memory layout / asking for help with zig fmt
2. @joachimschmidt557: comptime bitwise operations
3. @Luukdegram: Small wasm demo (importing js functions and calling zig functions from js)
4. @nektro: reconsider length between release cycles

## 2021-01-28
### Agenda

1. ~~@nektro: reconsider length between release cycles~~
2. @andrewrk: experimenting with a different memory layout for AST, with the goal of less memory usage and faster perf. If it is successful, the same strategies can be applied to ZIR and TZIR.

## 2021-01-21

### Agenda

1. @joachimschmidt557: How exitlude jump relocations are implemented in the ARM backend.
2. @andrewrk: (quick announcement) temporarily regressing .zir parsing/emission in order to rework it to better fit into the design
3. @Snektron: Discuss SPIR-V architecture definitions.
4. @max: Demo register allocation system
5. ~~@kubkon: Inside the linker: what is a `TextBlock` and how do we manage the free-lists?~~
6. @kubkon: Pushing stage2 to its limits - demo!
7. @andrewrk: Pitching my grand plan to make astgen emit ZIR that handles result locations optimally and solves cases like [this](https://gist.github.com/FireFox317/21a0109030491b3a81fe579b0052fe40). Goal is clean, maintainable compiler code with no hacks, as well as avoiding dead stores in the common cases, even in debug builds.
8. ~~@kubkon: if time permits, TLS + LLVM + Apple Silicon == no stack traces in stage1. More context [here](http://www.jakubkonka.com/2021/01/21/llvm-tls-apple-silicon.html).~~

## 2021-01-14

### Agenda

1. @Luuk: Refactoring Wasm backend in stage2
2. @joachimschmidt557: [#7187](https://github.com/ziglang/zig/issues/7187)
3. @joachimschmidt557: Questions about `reuseOperand` function in `codegen.zig`
4. @FireFox317: Demo of the LLVM backend in stage2
5. @kubkon: Fancy that! -> [#7781](https://github.com/ziglang/zig/pull/7781)

## 2021-01-07

### Agenda

1. @andrewrk: It's time to start tackling comptime performance in stage2. Here's a motivating test case: https://clbin.com/VdzX5
2. @andrewrk: Windows CI is no longer able to build zig1.obj (using stage1) due to OOM. We need to figure out how to make the CI green.
3. @andrewrk: [#7296](https://github.com/ziglang/zig/issues/7296)
4. @joachimschmidt557: duplication function parameters onto the stack (for debugging purposes etc.)
5. @kubkon: demo of extern fn linking in Mach-O.

## 2020-12-22

### Agenda

1. @andrewrk will demonstrate `tracy` integration -- `tracy` is a performance profiling tool. Here's a link to Andrew's old live coding session about it: [youtube.com](https://youtu.be/c0LRjqgtKS0).
2. @kubkon asked how we will handle `threadlocal` as a keyword. How is it currently handled in stage1 with LLVM?
3. @kubkon asked what is currently missing in stage2 to get the standard basic binary like

```zig
const std = @import("std");

pub fn main() void {
    std.log.info("Hello, world!", .{});
}
```

to successfully compile?

4. @andrewrk will discuss the new thread pool landing in `master`: [ziglang/zig#7462](https://github.com/ziglang/zig/pull/7462).
5. @kubkon made a couple of major changes to how we handle Mach-O in stage2 in the sense that, much like for Elf, we preallocate space for various segments and sections to aid in incremental linking. This allows us to skip rewriting `__LINKEDIT` segment every single time `__text` changes for instance. However, this means we break out from the convention set by other tools such as Apple's `ld` or LLVM's `ld64`. Additionally, this may imply that Apple's `codesign` tool will not validate the Mach-O binary due to certain hard-coded (wrong) assumptions about the structure of the Mach-O. Question here is: should we leave it as-is for Debug builds, and perhaps follow the convention in Release?

#### Meeting notes

1. A command that might be useful for Unix-like OSes:

```
nix-shell  -p cmake -p ninja -p pkg-config -p glfw -p freetype -p capstone -p tbb -p gtk3-x11
```

2. Thread-local is delegated to an appropriate LLVM API function. @kubkon demonstrated the origin of the question being a failing libstd test on `aarch64-macos-gnu` to do with declaring a global variable as thread-local. After some live debugging, we concluded that both `clang` and `zig` generate almost identical LLVM IR for it, and @kubkon later (offline) verified that when we swap LLD for Apple's system linker, the problem disappears. @andrewrk also mentioned that he posted a question on LLVM-dev mailing list inquiring what the chances are of `aarch64` MachO2 backend being ready for LLVM 12 release, and got a disheartening reply stating that "it's ready when it's ready". So yet another point in favour of ditching LLD and putting more effort into our own linker(s).

3. @andrewrk had a stab at answering that one and the answer is that we've got quite a few major hurdles to go over before that's done
   including `comptime`, panic handlers, etc. Specifically for Mach-O however, a good first achievable milestone would be to add support for
   `extern` in stage2 and link with `libSystem` functions instead of hand-writing `print` and `exit` in pure assembly as it is done now.
   
   EDIT: the downstream tracking issue: [ziglang/zig#7527](https://github.com/ziglang/zig/issues/7527)

4. TODO @andrewrk could you write this one up as I AFK when you were discussing it?

5. The entire point boils down to whether we should worry about the generated Mach-O binaries be compatible with Apple's `codesign` tool, and the short answer is no. @andrewrk suggested that we implement the incremental linker in the most optimal and yet correct way (which BTW is not what Apple assumes in their `codesign` or more specifically `codesign_allocate` tool), and see if our users are OK with it, and perhaps, with a bit of luck, have Apple fix their incorrect assumptions in `codesign_allocate.

## 2020-12-17

### Agenda

#### 1. Simplify translate-c implementation by introducing a new pseudo-ast data structures.
See [#6710](https://github.com/ziglang/zig/issues/6710) for more context. This discussion point is championed by @andrewrk.

#### 2. Middle-level IR.
It seems like this could be modeled as a sequence of lowering and optimization passes.  So something like
parse to ast -> comptime and semantic analysis -> runtime semantic IR -> backend agnostic optimizations -> backend -> backend-specific IR -> backend specific optimizations -> assembly.

The individual backends could do multiple passes of lowering followed by optimization, some of which are shared.  So you could make a "register machine"-specific IR and passes, and a "stack machine"-specific IR and passes, and each backend could choose to run the semantic IR through one of those shared lowering + optimization processes before doing really specific backend optimizations.

The problem with the value type you showed is neatly solved by this.  The problem is that you need to combine some low-level state about registers with high-level semantic state, because you're not able to go directly to assembly.  This solves that by introducing a new IR ("register machine IR") which gets produced from the semantic IR.  The code that produces this IR can be shared between any register machine backends, and then they can use the results in different ways to generate machine code.
Backends which aren't register machines, like wasm or the JVM, can do something different with the optimized semantic IR to produce their code.  And since the semantic IR doesn't have any machine specifics in it, there aren't any problems with incompatibility.

This discussion point is championed by @SpexGuy.

#### 3. LLVM-backend for the self-hosted.
Will include a demo. This discussion point is championed by @FireFox317.

### Meeting notes

meeting minutes:
 * 6710 is accepted 
 * @Specs_guy 's middle level IR idea is good and we're gonna try to do it
 * @FireFox317 showed an impressive LLVM backend demo, looking forward to merging that. We discussed how it would integrate into the rest of the compiler design
 * @Vexu brought up 7146. I'm gonna review it, ponder it, and then we're going to do a 1:1 follow up discussion to figure stuff out, specifically regarding how to do incremental compilation


re: @Specs_guy idea,  it might be a nice idea to learn about https://mlir.llvm.org/ as inspiration. we're not going to depend on it because that would be yet another dependency on a c++ llvm project thing, but it's tackling the same problem, so it could have some important ideas for us to consider


## 2020-12-10

### Agenda

* Identify everything that is linking-dependent codegen and linking-independent codegen
    * Linking-dependent: function calls, access to global variables(?)
    * Linking-independent: everything else ?
* Clean distinction between architecture-independent stuff and architecture-specific stuff
    * Encourage deduplication of code
    * Make it very contributer-friendly to add and improve backends
* `MCValue` needs to be completely architecture-dependent
    * Some entries can be considered universal and architecture-independent (embedded_in_code, memory)
    * Entries that are not universal: Register (no registers, general purpose, floating point, etc.) and more
    * `MCValue{ .memory = ... }` is used to populate and reference the GOT via absolute addressing (e.g., `movabsq` on `x86_64`); however, for platforms that require PIE such macOS, we need to fallback to RIP-relative/PC-relative addressing, therefore should `.memory` be re-used for PIEs or should we come up with an additional/alternative to it?
* Consider supporting architechtures without general purpose registers (stack-based, C backend)
* `codegen.zig` should not contain **any** architecture-dependent code, abstract only over stuff that is shared in **all** backends
* What should belong into semantic analysis, what should belong into codegen?
    * Architecture-independent optimizations (arithmetic expressions (strength reduction), `unreachable` optimizations, constant propagation, dead code elimination, loop unrolling, etc.): after sema and before codegen?
    * Generating checks for overflow in debug builds: sema or codegen?
* *(not technically 100% codegen)* What optimizations do we want to include?
    * e.g. ARM: conditional instructions

### Meeting Notes

For the kickoff meeting, the agenda was largely driven by @joachimschmidt57 who came up with this brilliant discussion write-up you can find above. Below you can find a summary of the discussion for each set of bullets from the agenda.

---

> * Identify everything that is linking-dependent codegen and linking-independent codegen
>    * Linking-dependent: function calls, access to global variables(?)
>    * Linking-independent: everything else ?

Linking and codegen are pretty tightly knit together (to better facilitate incremental linking). @joachimschmidt57 suggests we should split the linking-dependent from linking-independent bits in `codegen.zig`. Some examples:
* function calls - are always linking-dependent
* arithmetic calls - should be linking-independent

Consensus here was that (in the long run) we should audit the codegen/linking codebase and identify everything that is linking-dependent. This should help us come up with a plan on how to pull shared bits together and reorganise the codebase in general.

---

> * Clean distinction between architecture-independent stuff and architecture-specific stuff
>    * Encourage deduplication of code
>    * Make it very contributer-friendly to add and improve backends
> * `MCValue` needs to be completely architecture-dependent
>    * Some entries can be considered universal and architecture-independent (embedded_in_code, memory)
>    * Entries that are not universal: Register (no registers, general purpose, floating point, etc.) and more
>    * `MCValue{ .memory = ... }` is used to populate and reference the GOT via absolute addressing (e.g., `movabsq` on `x86_64`); however, for platforms that require PIE such macOS, we need to fallback to RIP-relative/PC-relative addressing, therefore should `.memory` be re-used for PIEs or should we come up with an additional/alternative to it?
> * Consider supporting architechtures without general purpose registers (stack-based, C backend)
> * `codegen.zig` should not contain **any** architecture-dependent code, abstract only over stuff that is shared in **all** backend

@joachimschmidt57 suggested we should move architecture-specific codegen bits (where appropriate) into their own `codegen` modules, for instance, ARM64 codegen-specific functionality would land in `codegen/aarch64.zig`. The general idea here would be for each architecture to have their own version of `MCValue`. One motivation for this is the fact that ARM64, in addition to general-purpose register, also has multiple special-purposed floating-point registers. Furthermore, we should also support register-free architecture and the current version of `MCValue` makes it somewhat clunky since register-free architectures will simply panic/error out on certain `MCValue` fields when switching over them, etc. @andrewrk supports this premise, however, as also pointed out by @ifreund and @luukdegram, we should be careful not to optimise prematurely since we haven't advanced the stage2 enough yet to have explored what `MCValue` would require for each architecture. The timing for this is perfect since @luukdegram only recently refactored Wasm32 to use the common codepath as other architectures: [ziglang/zig#7321](https://github.com/ziglang/zig/pull/7321).

---

> * What should belong into semantic analysis, what should belong into codegen?
>     * Architecture-independent optimizations (arithmetic expressions (strength reduction), `unreachable` optimizations, constant propagation, dead code elimination, loop unrolling, etc.): after sema and before codegen?
>     * Generating checks for overflow in debug builds: sema or codegen?
> * *(not technically 100% codegen)* What optimizations do we want to include?
>    * e.g. ARM: conditional instructions

We should identify which stuff goes in semantic analysis (sema) and which in codegen. For example, generating checks for overflows. @andrewrk said it should go in the sema.

Furthermore, @andrewrk said the plan for this is as follows:
1. first big step, make a simple, maintainable compiler that passes all the tests
2. optimise
3. design direction is for less memory usage in opcodes; ZIR can be architecture-dependent by design if it solves a particular problem for a particular architecture/platform
4. comptime knows the target architecture and immitates it -> sema is very aware of the target

@joachimschmidt57 asked if the optimisations should go after sema but before codegen in the pipeline. @andrewrk said that we don't have any yet they indeed should go between sema and codegen; optimisations take typed IR instructions as the input, and give back optimised typed IR instructions as the output. What optimisations do we want to include? For debug builds, none. No LLVM but Release mode (this would be our self-hosted backend) -> our hand-crafted optimisations. This might be a totally different codegen path. It is important to note that the focus for 1.0 release is to have self-hosted in Debug, and LLVM-backed in Release.

If we figure out some good optimisations, it would be beneficial to insert them between codegen and the LLVM backend; e.g., async functions etc.
