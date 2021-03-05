Here is a sample shell.nix for building/developing zig.
```nix
{ pkgs ? import <nixpkgs> { } }:

pkgs.mkShell {
  hardeningDisable = [ "all" ];
  buildInputs = with pkgs; [
    cmake
    gdb
    clang
    llvmPackages_11.clang-unwrapped
    llvm_11
    lld_11
    ninja
    qemu
  ];
}
```
the `hardeningDisable` part is crucial otherwise you will get compile errors.