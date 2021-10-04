Here is a sample shell.nix for building/developing zig.
```nix
{ pkgs ? import <nixpkgs> { } }:

pkgs.mkShell {
  hardeningDisable = [ "all" ];
  buildInputs = with pkgs; [
    cmake
    gdb
    ninja
    qemu
  ] ++ (with llvmPackages_13; [
    clang
    clang-unwrapped
    lld
    llvm
  ]);
}
```
The `hardeningDisable` part is crucial, otherwise you will get compile errors.