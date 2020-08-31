Note: Official builds of master branch and releases are [available for download](https://ziglang.org/download/).

See [Repology](https://repology.org/project/zig/versions) for an overview of which package management systems and operating systems Zig has already been packaged for, and what the current packaged version is.

[![Packaging status](https://repology.org/badge/vertical-allrepos/zig.svg)](https://repology.org/project/zig/versions)

## Homebrew

### Latest tagged release
```
brew install zig
```

### Latest build from Git master branch
```
brew install zig --HEAD
```

## MacPorts

```
port install zig
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

## Ubuntu or Debian (using apt)

After you've completed the [repository configuration](https://github.com/dryzig/zig-debian/blob/master/README.md), installation is as easy as the familiar:

```
sudo apt install zig
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
pacman -S zig
```

[zig-static AUR](https://aur.archlinux.org/packages/zig-static/) - This package uses the official ziglang.org static build instead of building against source. In the event that the llvm version in [extra] is not up to date with the latest version used by zig, this package can be used since it has no llvm dependency.

## Gentoo
```
emerge dev-lang/zig
```

### Latest build from Git master branch
```
emerge '=dev-lang/zig-9999'
```

## Void Linux

```
xbps-install -Su zig
```

## DragonFlyBSD (using [ravenports](http://www.ravenports.com/))

```
ravensw install zig-single-standard
```

