## Traditional approach

Here is a sample `shell.nix` for building/developing zig.
```nix
with import <nixpkgs> {};

pkgs.mkShell {
  nativeBuildInputs = with pkgs; [
    cmake
    gdb
    ninja
    qemu
    wasmtime
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

  outputs = inputs: inputs.flake-utils.lib.eachDefaultSystem (system:
    let
      pkgs = inputs.nixpkgs.legacyPackages.${system};
    in
      {
         devShell.${system} = pkgs.mkShell {
           nativeBuildInputs = with pkgs; [
             cmake
             gdb
             ninja
             qemu
             wasmtime
           ] ++ (with llvmPackages_13; [
             clang
             clang-unwrapped
             lld
             llvm
           ]);

           hardeningDisable = [ "all" ];
         };
      }
  );
}
```

## Notes for macOS users

If using macOS, you will need to [build llvm yourself](https://github.com/ziglang/zig/wiki/How-to-build-LLVM,-libclang,-and-liblld-from-source) and use the system's clang to build zig. This is possible using the sample `shell.nix` below.

```nix
with import <nixpkgs> { };

pkgs.mkShellNoCC {
  nativeBuildInputs = with pkgs; [ cmake gdb ninja qemu wasmtime ];

  hardeningDisable = [ "all" ];
}
```