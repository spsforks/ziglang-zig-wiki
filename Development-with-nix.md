## Traditional approach

Here is a sample `shell.nix` for building/developing zig.
```nix
let
  nixpkgs = builtins.fetchTarball {
    # nixpkgs-unstable (2021-10-28)
    url = "https://github.com/NixOS/nixpkgs/archive/22a500a3f87bbce73bd8d777ef920b43a636f018.tar.gz";
    sha256 = "1rqp9nf45m03mfh4x972whw2gsaz5x44l3dy6p639ib565g24rmh";
  };
in
{ pkgs ? import nixpkgs { } }:

pkgs.mkShell {
  nativeBuildInputs = with pkgs; [
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

  hardeningDisable = [ "all" ];
}
```
The `hardeningDisable` part is crucial, otherwise you will get compile errors.

## Flake

Alternatively, you can use this sample `flake.nix`:

```nix
{
  description = "A flake for Zig development";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixpkgs-unstable";
    flake-utils.url = "github:numtide/flake-utils";
    flake-compat = {
      url = "github:edolstra/flake-compat";
      flake = false;
    };
  };

  outputs = inputs:
    let
      system = "x86_64-linux";
      pkgs = inputs.nixpkgs.legacyPackages.${system};
    in
      {
         devShell.${system} = pkgs.mkShell {
           nativeBuildInputs = with pkgs; [
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

           hardeningDisable = [ "all" ];
         };
      };
}
```