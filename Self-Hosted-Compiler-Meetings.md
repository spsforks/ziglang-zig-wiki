Once per week, organized by @kubkon. Send him a message to get your agenda item for next week added.

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
