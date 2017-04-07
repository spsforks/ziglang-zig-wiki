This page is documenting what I did to get automated Windows testing set up, for my own records.

 * Get a dedicated server that has virtualization hardware features. Virtualized servers often cannot run virtualbox.

```
VBoxManage createvm --name "Windows7Ultimate" --ostype Windows7_64 --register
VBoxManage modifyvm "Windows7Ultimate" --memory 4096
VBoxManage createhd --filename "Windows7.vdi" --size 30000
VBoxManage storagectl "Windows7Ultimate" --name "IDE Controller" --add ide --controller PIIX4
VBoxManage storageattach "Windows7Ultimate" --storagectl "IDE Controller" --port 0 --device 0 --type hdd --medium "Windows7.vdi"
VBoxManage storageattach "Windows7Ultimate" --storagectl "IDE Controller" --port 0 --device 1 --type dvddrive --medium /root/Windows7-64bit-Ultimate.iso
VBoxManage modifyvm "Windows7Ultimate" --vrde on
VBoxManage modifyvm "Windows7Ultimate" --vrdeport 3389
# get the extension pack that matches virtualbox version
VBoxManage extpack install Oracle_VM_VirtualBox_Extension_Pack-xxxxx.vbox-extpack
VBoxManage setproperty vrdeextpack "Oracle VM VirtualBox Extension Pack"
VBoxManage startvm "Windows7Ultimate" --type headless
```

Now RDP and install Windows.

```
rdesktop host:3389
```

 * Install guest additions.

```
sudo apt install virtualbox-guest-additions-iso
VBoxManage storageattach "Windows7Ultimate" --storagectl "IDE Controller" --port 0 --device 1 --type dvddrive --medium /usr/share/virtualbox/VBoxGuestAdditions.iso
```

 * Run Windows Update (wait for days)
 * http://www.msys2.org/ - install it then follow directions on that site to update pacman

```
-pacman -S git gcc cmake make tar python
```

Install LLVM libraries

```
cd ~/
wget http://releases.llvm.org/4.0.0/llvm-4.0.0.src.tar.xz
tar xvf llvm-4.0.0.src.tar.xz
cd llvm-4.0.0.src
mkdir build
cd build
cmake ..  -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local
make install
```

Install clang libraries

```
cd ~/
wget http://releases.llvm.org/4.0.0/cfe-4.0.0.src.tar.xz
tar xvf cfe-4.0.0.src.tar.xz
cd cfe-4.0.0.src
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local
make install
```

Install LLD libraries

```
cd ~/
wget http://releases.llvm.org/4.0.0/lld-4.0.0.src.tar.xz
tar xvf lld-4.0.0.src.tar.xz
cd lld-4.0.0.src
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/local -DCMAKE_PREFIX_PATH=$HOME/local
make install
```

```
git clone https://github.com/andrewrk/zig/
cd zig
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$(cwd) -DZIG_LIBC_LIB_DIR=/usr/lib -DZIG_LIBC_INCLUDE_DIR=/usr/include -DZIG_LIBC_STATIC_LIB_DIR=/usr/lib/gcc/x86_64-pc-msys/6.3.0/
```