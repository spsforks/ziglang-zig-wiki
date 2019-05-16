Note: Official builds of master branch and releases are [available for download](https://ziglang.org/download/).

## Homebrew

### Latest tagged release
```
brew install zig
```

### Latest build from Git master branch
```
brew install zig --HEAD
```

## Ubuntu (using [snap](https://snapcraft.io/zig))

### Latest tagged release
```
snap install zig --classic --beta
```

### Latest build from Git master branch
```
snap install zig --classic --edge
```

## NixOS

```
nix-env -i zig
```

## Windows (using [scoop](http://scoop.sh/))

```
scoop install ziglang
```

## Windows (using [chocolatey](https://chocolatey.org))

```
choco install zig
```

## Arch Linux

```
pacman -Sy zig
```

[zig-static AUR](https://aur.archlinux.org/packages/zig-static/) - This package uses the official ziglang.org static build instead of building against source. In the event that the llvm version in [extra] is not up to date with the latest version used by zig, this package can be used since it has no llvm dependency.