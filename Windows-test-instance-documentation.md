This page is documenting what I did to get automated Windows testing set up, for my own records.

 * Get a dedicated server that has virtualization hardware features. Virtualized servers often cannot run virtualbox.

```
VBoxManage createvm --name "Windows10" --ostype Windows10_64 --register
VBoxManage modifyvm "Windows10" --memory 4096
VBoxManage createhd --filename "Windows7.vdi" --size 50000
VBoxManage storagectl "Windows10" --name "IDE Controller" --add ide --controller PIIX4
VBoxManage storageattach "Windows10" --storagectl "IDE Controller" --port 0 --device 0 --type hdd --medium "Windows7.vdi"
VBoxManage storageattach "Windows10" --storagectl "IDE Controller" --port 0 --device 1 --type dvddrive --medium /root/Windows10-64bit.iso
VBoxManage modifyvm "Windows10" --vrde on
VBoxManage modifyvm "Windows10" --vrdeport 3389
# get the extension pack that matches virtualbox version
VBoxManage extpack install Oracle_VM_VirtualBox_Extension_Pack-xxxxx.vbox-extpack
VBoxManage setproperty vrdeextpack "Oracle VM VirtualBox Extension Pack"
VBoxManage startvm "Windows10" --type headless
```

Now RDP and install Windows.

```
rdesktop host:3389
```

 * Install guest additions.

```
sudo apt install virtualbox-guest-additions-iso
VBoxManage storageattach "Windows10" --storagectl "IDE Controller" --port 0 --device 1 --type dvddrive --medium /usr/share/virtualbox/VBoxGuestAdditions.iso
```
 * Install Visual Studio, C++ Desktop Development
 * Download LLVM, Clang, and LLD source
 * Install 7zip http://www.7-zip.org/ and extract them all
 * Install Python https://www.python.org/
 * Install CMake https://cmake.org/

 * Open Folder -> llvm source folder
 * CMake -> Change CMake Settings -> CMakeLists.txt
   * `"cmakeCommandArgs": "-DPYTHON_EXECUTABLE=C:\\Users\\andy\\AppData\\Local\\Programs\\Python\\Python36-32\\python.exe",`
   * Save the file
 * Wait for a long time so that "Build" becomes an option in the right click menu of CMakeLists.txt
 * Click Build
 * Right click CMakeLists.txt, click install
 * Find the folder, move the install folder to c:\deps\llvm-4.0.0

 * Open Folder -> clang source folder
 * CMake -> Change CMake Settings -> CMakeLists.txt
   * `"cmakeCommandArgs": "-DPYTHON_EXECUTABLE=C:\\\\Users\\\\andy\\\\AppData\\\\Local\\\\Programs\\\\Python\\\\Python36-32\\\\python.exe -DLLVM_CONFIG=C:\\deps\\llvm-4.0.0\\bin\\llvm-config.exe",`
   * save the file
 * right click a different file then right click CMakLists.txt, click build, not sure what makes this menu item available
 * get a cryptic error message, give up

 * Open Folder -> lld source folder
 * CMake -> Change CMake Settings -> CMakeLists.txt
   * `"cmakeCommandArgs": "-DLLVM_CONFIG_PATH=C:\\deps\\llvm-4.0.0\\bin\\llvm-config.exe",`
 * right click CMakLists.txt, click build
 * right click CMakLists.txt, click install
 * Find the folder, move the install folder to c:\deps\lld-4.0.0


 * Use Visual Studio to git clone https://github.com/andrewrk/zig
