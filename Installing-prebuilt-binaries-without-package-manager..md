Once you download the tarball just unzip it then just run `ln -s /path/to/zig/folder/zig /usr/bin/zig
` as root

1. Download and install curl,tar,xz. It is shipped with Git on Windows. With the shell, `cd` to the folder you wish zig to reside.
2. Download and extract zig with
```sh
set -eou  # error out, if anything fails
curl -sS -O LINKTOTARBALL #https://ziglang.org/download/
sha256sum HASHONWEBSITE #https://ziglang.org/download/
tar -xvf TARBALL
```
3. Adjust the PATH of your shell to look for for zig with
```sh
#in your .bashrc or config file or your shell add something like (depends on your system)
PATH=$PATH:PATHTOZIGEXECUTABLE #you could use XDGBDS (Linux): PATH=$PATH:"${HOME}"/.local/bin/zig
```