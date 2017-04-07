This page is documenting what I did to get automated Windows testing set up, for my own records.

 * Get a dedicated server that has virtualization hardware features. Virtualized servers often cannot run virtualbox.

```
VBoxManage createvm --name "Windows7Ultimate" --ostype Windows7_64 --register
VBoxManage modifyvm "Windows7Ultimate" --memory 4096
VBoxManage createhd --filename "Windows7.vdi" --size 20000
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
 * Run Windows Update (wait for days)

