The meetings happen weekly, every Thursday at 19.00 UTC, on [this Discord server](https://discord.gg/gxsFFjE)

Edit this wiki to get your agenda item for next week added.

## 2022-03-17

1. @joachimschmidt557
    - remove `MCValue.embedded_in_code`
        - consensus: yes, do it
        - switch jump tables will be lowered using a different mechanism than `MCValue.embedded_in_code`
    - register allocation for floating point registers
        - Multiple `RegisterManager`s or one unified?
        - We also need to keep vector registers in mind
        - Try multiple `RegisterManager`s approach for now
2. @kubkon
    - demo of hot-code reloading on macOS
        - AArch64 backend still needs some work for stdlib `nanosleep`
        - ~~Requires sudo for now~~ Managed to bake in entitlements as explained in the follow-up blog post: https://www.jakubkonka.com/2022/03/22/hcs-zig-part-two.html
            - needs to open Mach IPC port
            - LLDB compiled from source can do this without sudo using entitlements
            - we will do this too
        - Requires manual pkill of child process for now
        - Also works with ASLR enabled (we kind of act like a dynamic linker)
        - Blog post on Jakub's homepage: http://www.jakubkonka.com/2022/03/16/hcs-zig.html
3. @Snektron
    - Showcase some spir-v progress
        - stage2 can compile a SPIR-V shader which can then be executed on the GPU by a Zig program which was also compiled with stage2
        - improvements to SPIR-V inline assembly will be taken into account when overhauling inline assembly
4. @andrewrk
    - quick progress update and asking people with open PRs to please rebase because of the recent CI failure

## 2022-03-10

1. @andrewrk
   - let's figure out how we want Sema to emit debug information for local variables that works well for the LLVM backend as well as the native backends and wasm backend.
       - Mapping of variable names to locations (stack, register)
       - Add 2 more AIR instructions: "debug pointer" and "debug value" (corresponding to the LLVM debug instructions)
       - We should also emit spills (register -> stack offset and vice versa) in DWARF
       - There was still uncertainty whether 2 instructions were necessary or 1 is enough
       - Andrew will implement this for LLVM (and possibly reduce it to 1 extra instruction)
2. @kristoff-it
    - Showcase current state of Autodoc, briefly explain implementation layout & main techniques adopted, invite others to contribute. [Video trailer](https://clips.twitch.tv/NastySmallDonkeyM4xHeh-MRPWeTcGhj0ezOlV).
        - [Slides](https://docs.google.com/presentation/d/1I7ofH3Q1dZdYjAymDNUJK8DzZeO7gpt1eAFDZpcEIRA/edit?usp=sharing)
        - Autodoc can render `std.ascii` already
        - Correct terminology: Autodoc = system that generates docs, Autodocs = the generated docs
        - High-Level Design
            - Single-Page Application, search embedded in page
            - Optimise for memory usage
        - Additional features (not present in stage1 autodoc)
            - toggle for internal mode (shows private decls)
            - Sources section (syntax-highlighted Zig code)
            - Guides section (hand-written docs)
        - Autodoc uses ZIR
            - for complex expressions which cannot be trivially evaluated, Autodoc uses `ComptimeExpr`
            - Possibility: Wait for Sema to complete and use more info from there; map Sema information to `ComptimeExpr`s
            - `ArrayList`: When both `ArrayList(u8)` and `ArrayList([]const u8)` are used, show these example instatiations
        - Vision for Guides
            - searchable
            - maybe generated from folder with Markdown files?
            - figure out vision later

## 2022-03-03

1. @joachimschmidt557
    - runtime safety checks in Sema without generating too much extra machine code
        - related issues: https://github.com/ziglang/zig/issues/10248 https://github.com/ziglang/zig/issues/84
        - way forward for arithmetic operations: Sema will generate an `add_with_overflow` instruction for add operations when runtime safety is on, else will emit `add` (for subtractions, multiplications similar)
        - as `@addWithOverflow` and co. return a tuple, this enables the codegen backends to specialise on this: For e.g. ARM, `@addWithOverflow` can return an MCValue consisting of a register and the condition flags (including overflow flag). The overflow check can then be lowered to check the condition flags.
        - We will introduce optimizations to lowering of `cond_br` in codegen. If one branch ends with unreachable, we can remove the need for one branch instruction as control flow will never leave that branch again. Checking whether a branch ends in unreachable is trivial: we just check the last instruction in the branch.
        - Why Sema generates the panic: This enables us to skip emitting the panic handler if it is never called.
        - Related: For `try`, Sema can generate conditional branches with some sort of hint that the branch for the error case can jump to the end of the function (or similar) and have the non-error path (which should be the hot path) in the machine code as "uninterrupted" as possible.
2. Type Coercion in AstGen vs in Sema
```zig
const a = true;
const b = if (a) 1 else 2;
```
At the moment, this generates a coercion to bool in AstGen which should be moved to Sema.

3. https://github.com/ziglang/zig/issues/11046
    - one of the last things preventing `std.debug.dumpStackTrace`
    - @andrewrk wants to tackle this

## 2022-02-10

1. @joachimschmidt557
    - codegen: third rewrite of lowering mechanics for binary operations: showcase and feedback
2. @kubkon
    - codegen+linker: lowering of slices: showcase
    - codegen: truncation, signed, non-power-of-two: showcase and feedback
    - codegen: PIE mechanics: showcase and feedback
    - linker: refactoring of `LinkBlock` abstraction
3. @topolarity
    - sema: Does "Value.read/writeToMemory" need to respect the ABI size of exotic (non-power-of-two) types? Is memory required to be sign-extended to the full buffer size? (Context: Trying to fix an observed bug in `@bitCast` for exotic integers, where the sign bit is ignored)
4. @andrewrk
    - Pitch my SegmentedList idea
5. @agni
    - :^)

## 2022-01-27

1. @g_w1
    - For https://github.com/ziglang/zig/issues/7923 , what should the parser type of the ident be?
      We don't want to allow `test u32 {` or `test @as(...)` for example, but do we want `test @This() {`? What should the parser do vs astgen vs sema?

## 2022-01-20

1. @andrewrk
   - Should we move the concept of byval/byref from the backend to the frontend? In other words, the frontend would ask the backend if a given type was byval or byref, and avoid using the byval instructions, such as elem_val, for types that are byref. This would make Sema emit allocations rather than codegen backends having to find out (possibly too late) that they needed more allocations for byval stuff.
2. @Luukdegram
    - Moving and enable/disable behavior tests.

## 2022-01-13
1. @andrewrk
    - Getting help from @Luukdegram on the wasm backend to troubleshoot #10582
2. @joachimschmidt557
    - Register allocation stuff again (yay)
        - API change
        - scratch register for Emit pass
3. @kubkon
    - proposal: storing pointers in linear memory: dealing with moved atoms/decls in the linker via relocation mechanism rather than in codegen


## 2021-12-23
1. @gwenzek
    - Heterogeneous programming support #10397
2. @Luukdegram
    - Address spaces in Wasm. Solving #4866

## 2021-12-15

1. @mattnite
    - BTF: getting useful BTF info when compiling for BPF, see if there are other ideas for llvm intrinsics.

## 2021-11-25

1. @joachimschmidt557
    - Register allocation: maybe just use the stack for now and focus on getting more tests passing and then design a generic and efficient register allocator (credits for that idea: Loris and Meghan)
2. @Luukdegram
    - ABI implementations: Ensuring correctness between backends (llvm vs ours).
    - (Info for the curious reader: https://github.com/WebAssembly/tool-conventions/blob/main/BasicCABI.md)

## 2021-11-18

 1. @kubkon
    - callee preserved registers on x86_64 - do we reserve `rax` for params and return values only, or is a block free to use after pushing it on the stack?
    - related issue - https://github.com/ziglang/zig/pull/10140
    - for register allocator - do we track register liveness?
 2. @jmc
    - signal boost for the new wiki pages [Contributing to Stage 2](https://github.com/ziglang/zig/wiki/Contributing-to-Stage-2) and [C backend signup sheet](https://github.com/ziglang/zig/wiki/C-Backend-Behavioral-Tests-signup-sheet)

## 2021-11-11

Meeting will be skipped due to [Handmade Seattle](https://www.handmade-seattle.com/).

## 2021-11-04

1. @Luukdegram
    - Proposal and reasoning for emitting MIR in the wasm backend.
        - Supporting document: https://docs.google.com/document/d/1JEUe5ivuMHmIY8ehkVHihWIq9bFsvsB0IuVbrLp4Mak/edit?usp=sharing
2. @kubkon
    - progress on MIR transition of x86_64 backend
3. @joachimschmidt557
    - MIR branch lowering algorithm for AArch64
4. @andrewrk
    - showing off ziglang.org/perf and looking for contributions to add benchmarks to gotta-go-fast

## 2021-10-28

1. @joachimschmidt557
    - Progress on MIR transition of AArch64 backend
    - MIR branch lowering
2. @The-King-of-Toasters
    - Discussion on how to work in the event loop code.
        - End goal: add support for Solaris event ports.
        - Easy win: merge the openbsd for initOsData with the rest of the KQueue OSs.
        - Possibly shadow @kprotty to get my head around the codebase.
    - Adding seccomp definitions to `os.linux`
        - Requires filling in definitions for "classic" BPF that many OSs use, split out into own namespace?
        - Seccomp programs load members from the `seccomp_data` structure (see `<linux/seccomp.h>`) using offsets.
          A common idiom for seccomp programs in C is `offsetof(seccomp_data, arg[n])`, where arg is an array of u64s.
          I have a workaround for this, but it would be nice to have this added for parity with C (would be interested in doing this).
3. @andrewrk
   - progress demo & roadmap

## 2021-10-21

1. @joachimschmidt557
    - Register allocation in a separate pass: discussion
2. @paezao
    - Mentoring for https://github.com/ziglang/zig/issues/9185
3. @kubkon
    - Demo and review of MachO's linker state dumping mechanism: https://github.com/ziglang/zig/pull/9985
4. @moosichu
    - Request for clarification on zig ar implementation: https://github.com/moosichu/zar, https://github.com/ziglang/zig/issues/9828
      Steady progress is being made (big thanks to @iddev5), but as features get fleshed out it would be good to have an idea of a final end-goal.
       - What does it need to be considered ready for upstream integration, beyond coverage of the llvm-ar features?
       - Some behavioural differences seem to be desired from llvm-ar (in a cross-compilation setting), so it would be good to know where it struggles so we can address that.
       - Because the goal is to be a drop-in replacement for an existing program. Is that the only goal, or are there others? Should that be a compatibility mode or how the program behaves by default?
       - I also have some ideas for testing, and I want to make sure they make sense (depends on if we can use llvm-ar through zig as well).
       - Also happy to answer any questions/concerns anyone might have.
 

## 2021-10-14

1. @kubkon
   - progress on incremental Mach-O linker
   - progress on zld ELF/x86_64
2. @andrewrk
   - brainstorming on MIR
3. @andyfleming
   - help on appropriate way to run test subset (compile_errors.zig)
     - Issue: https://github.com/ziglang/zig/issues/9203
     - WIP: https://github.com/ziglang/zig/compare/master...andyfleming:fix-9203?expand=1
4. @nektro
   - Q on building with llvm13
   - Q on https://github.com/ziglang/zig/commit/b0f80ef0d5517b5ce8ba60dfae6dd986346f8d4c

## 2021-09-23

1. @joachimschmidt557
    - Thoughts on https://github.com/ziglang/zig/issues/9798

## 2021-09-16

1. @kubkon
   - demo of the MachO linker after merging traditional with incremental codepaths - https://github.com/ziglang/zig/pull/9676
   - what's next for MachO?

## 2021-09-02

1. @g-w1
   - question about ++ and ** at runtime
   - maybe plan9 debuginfo demo
2. @kubkon
   - update from the linker depths - https://github.com/ziglang/zig/pull/9676
   - discussion of current limitations for self-hosted for PIE targets
   - demo of self-hosted linking Zig with compiled C object file
3. @nektro
   - I submitted my first stage2 PR :)
   - Then I submitted 3 more
   - Not super demo-able, just excited. I can overview the bugs they fixed though

## 2021-08-26
1. @andrewrk
   - short "how to start contributing" talk
   - talk about `usingnamespace`
   - stage2 progress report
   - show some stage2 debugging tips
   - here's a contributor friendly item: https://github.com/ziglang/zig/issues/9622
   - brief explanation about Value/Type tag() vs zigTypeTag()

## 2021-07-29
1. @andrewrk
   - Pitch to see if anyone wants to try deleting the "BoundFn" type from the language and compiler (both stage1 and stage2). [#9484](https://github.com/ziglang/zig/issues/9484)
   - Demo of recent progress on getting `zig test` to work for stage2.
   - Discussion: should we kill the "ref" AIR instruction?
2. @joachmschmidt557
   - compare flags issues in self-hosted codegen

## 2021-07-15
1. @andrewrk
   - Pitch to see if anyone wants to take on `zig objcopy` [#9261](https://github.com/ziglang/zig/issues/9261)
2. @jmc
   - https://github.com/ziglang/zig/issues/9257 was closed, but not sure it's "fixed", see discussion there
   - Is https://github.com/ziglang/zig/pull/9183 mergeable?

## 2021-07-08
1. @andrewrk
   - Demo some recent progress
   - Pitch to see if anyone wants to take on a stage1 memory task to help us accelerate stage2 work this release cycle
   - Pitch to see if anyone wants to take on a project to track compilation speed over time
   - Picking back up work on AIR memory layout, specifically with a design goal in mind of improving simplicity of codegen by putting all the constants up front so they can be possibly lowered to memory without jumps
   - Code review & merging of any stage2 open PRs during the meeting
2. @greenfork
   - Questions about linker, [relevant PR](https://github.com/ziglang/zig/pull/9293)
3. @kubkon
   - Demo some recent progress on linker rewrite - merging Zld with incremental MachO!

## 2021-06-24
1. @g-w1
    - Demo plan9 hello world!
2. @andrewrk
    - Brief explanation of how ast-check is now running even when using the stage1 compiler.
3. @greenfork
    - Ideas on tilde error printing, questions [PR](https://github.com/ziglang/zig/pull/9201), [why parent](https://github.com/ziglang/zig/blob/master/src/Module.zig#L1456)
4. @redj
    - https://github.com/ziglang/zig/pull/9029 vs @kubkon's ObjC support.
5. @jmc
    - Opinions on https://github.com/ziglang/zig/issues/9182?

## 2021-06-17
1. @g-w1
    - Demo plan9 linker.
    - Discuss more inline assembly to support hello world/show abi.

## 2021-06-10
1. @Vexu
    - I have some issues and prs that could be discussed to try to shrink the pr queue.
    - Maybe a quick demo of stage2 comptime vars?
2. @andrewrk
    - It's time to rework AIR Memory Layout. Is anybody interested in taking it on? I have a branch to get you started if you want.

## 2021-05-27
### Agenda
1. @Luukdegram
    - wasm proposals, versions, and how to handle them regarding compatibility. (i.e. multi-value returns)
2. @g-w1
    - showcase unused var compile error

## 2021-05-20
1. @Snektron
   - progress on SPIR-V
2. @kubkon
   - progress and Q&A on shipping WASI libc in Zig - [#8837](https://github.com/ziglang/zig/pull/8837)
   - what to do about `zig c++ -target wasm32-wasi`?
3. @andrewrk
   - demo some progress from whole-file-astgen
   - release plan for 0.8.0 - what is left, what to prioritise, etc.

## 2021-05-13
### Agenda
1. @kubkon
   - zld demo: stage1 bootstrapped with `zig-bootstrap` passes tests on macOS
   - Wanna run macOS tests but don't have a Mac? No probs! Zig's got you covered (at least partially): [#8760](https://github.com/ziglang/zig/pull/8760)
2. @andrewrk
   - progress on stage2-whole-file-astgen
   - native libc++
   - pitch the idea of test harness loading code at runtime

## 2021-04-29
### Agenda
1. @ifreund
   - std/build: change default install prefix from local zig-cache to . #8638

## 2021-04-22
### Agenda
1. @andrewrk
   - Demo whole-file-AstGen
   - Outline what a hot code reloading feature would look like

## 2021-04-15
### Agenda

1. @joachimschmidt557:
    - Register manager: Register classes
2. @amro:
    - Stage 1 C ABI struct parameters / return values
3. @andrewrk:
    - llvm12 update
    - demo work-in-progress whole-file-astgen and std.zig integration

## 2021-04-08
### Agenda

1. @andrewrk
    - Go over some contributor friendly stage2 issues, try to inspire some fresh faces to get involved :)
    - Demo recent progress on structs & enums
    - LLVM 12 update
2. @joachimschmidt557:
    - Questions about reusing stack locations (stage2 codegen)

## 2021-04-01
### Agenda
1. @joachimschmidt557:
    - Register allocation: Use function arguments from stack
    - Questions about compiler_rt
1. @g-w1
    - Showcase ast-based doc generator
    - Ask for feedback, specifically about design

## 2021-03-25
### Agenda
1. @andrewrk:
   - brief overview of the compiler pipeline
   - zir-memory-layout branch update
   - a plan for inline while, comptime code, etc in stage2
2. @nektro: plz explain stage1 ZigType + ZigValue

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
