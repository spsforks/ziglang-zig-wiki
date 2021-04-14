Here is a sample shell.nix for building/developing zig.
```nix
{ pkgs ? import <nixpkgs> { } }:

pkgs.mkShell {
  hardeningDisable = [ "all" ];
  buildInputs = with pkgs; [
    cmake
    gdb
    clang
    llvmPackages_12.clang-unwrapped
    llvm_12
    lld_12
    ninja
    qemu
  ];
}
```
The `hardeningDisable` part is crucial otherwise you will get compile errors.