# Settin up QEMU Development Environment

Install [kvm](https://wiki.archlinux.org/index.php/KVM) and [qemu](https://wiki.archlinux.org/index.php/QEMU).

Create a hard disk image file using. Following command will create  a  
```sh
qemu-img create ubuntu.img 5G
```

Install ubuntu 14.04 in the hard disk image we just created
```sh
qemu-system-x86_64 -enable-kvm -hda ubuntu.img -cdrom ~/ubuntu-14.04-desktop-amd64+mac.iso -boot d -m 2G
```

Get qemu source code from `git://git.qemu-project.org/qemu.git`

Compile qemu using make.

Run the compile qemu using
```sh
x86_64-softmmu/qemu-system-x86_64 -enable-kvm  -hda ../ubuntu.img -device virtio-balloon  -m 2G
```
