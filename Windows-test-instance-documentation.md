This page is documenting what I did to get automated Windows testing set up, for my own records.

 * Get a dedicated server that has virtualization hardware features. Virtualized servers often cannot run virtualbox.

```
VBoxManage createvm --name "Windows10" --ostype Windows10_64 --register
VBoxManage modifyvm "Windows10" --memory 4096
VBoxManage createhd --filename "Windows7.vdi" --size 40000
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
