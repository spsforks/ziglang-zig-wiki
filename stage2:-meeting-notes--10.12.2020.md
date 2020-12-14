# Agenda

### Thoughts on zig stage2 codegen

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

# Meeting notes

For the kickoff meeting, the agenda was largely driven by @joachimschmidt57 who came up with this brilliant discussion write-up you can find above.

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

@joachimschmidt57 suggested we should move architecture-specific codegen bits (where appropriate) into their own `codegen` modules, for instance, ARM64 codegen-specific functionality would land in `codegen/aarch64.zig`. The general idea here would be for each architecture to have their own version of `MCValue`. One motivation for this is the fact that ARM64, in addition to general-purpose register, also has multiple special-purposed floating-point registers. Furthermore, we should also support register-free architecture and the current version of `MCValue` makes it somewhat clunky since register-free architectures will simply panic/error out on certain `MCValue` fields when switching over them, etc. @andrewrk supports this premise, however, as also pointed out by @ifreund and @luukdegram, we should be careful not to optimise prematurely since we haven't advanced the stage2 enough yet to have explored what `MCValue` would require for each architecture. The timing for this is perfect since @luukdegram only recently refactored Wasm32 to use the common codepath as other architectures: [ziglang/zig#7321](https://github.com/ziglang/zig/pull/7321).