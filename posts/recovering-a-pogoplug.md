# [Recovering a PogoPlug](/recovering-a-pogoplug)

Several weeks ago I made a few untested `sshd` config changes and managed to lock myself out of my [PogoPlugv4](/pogoplugv4). Due to the PogoPlug not having a display port, I had to resort to editing the files on my MacBookPro.

The problem here is that my laptop is running x64, but the PogoPlug is running [ARM][]. So using Virtualbox wont work, it only supports x86 and x64. However, we can use [QEMU][] to _emulate_ a machine with ARM architecture.

This post assumes you're using OSX for your host computer and that you've installed your PogoPlug OS on a SATA drive.

If you're using Linux, replace `brew` with whatever package manager you have. These directions _might_ work as-is if you installed the PogoPlug OS on a USB drive.

## Dependencies

* [Homebrew][]
* [USB to SATA Adapter][] (_USB 3.0 optional_)

## How

Go install `brew`, then use it to install QEMU.

```
$ brew install qemu
```

Download an ARM OS image, kernel, and initrd.

```
$ wget https://people.debian.org/~aurel32/qemu/armel/debian_wheezy_armel_standard.qcow2
$ wget https://people.debian.org/~aurel32/qemu/armel/vmlinuz-3.2.0-4-versatile
$ wget https://people.debian.org/~aurel32/qemu/armel/initrd.img-3.2.0-4-versatile
```

Here are the hashes if you're that sort of person.

```
4b830c500591181e3af2d832da39f1ba  debian_wheezy_armel_standard.qcow2
ed7c39ec86e759240bdddd783248ed8b  initrd.img-3.2.0-4-versatile
3bdf3393243e65bd862b1398a494134a  vmlinuz-3.2.0-4-versatile
```

Plug the SATA drive in to the host machine using the adaptor. OSX might not recognize the drive, that's ok. Once it's plugged in, we need to find it's location.

```
$ diskutil list
```

Run QEMU with the USB drive attached. In this case `/dev/disk1s1` means `disk1` with partition `1`. This could be different for you.

```
$ sudo qemu-system-arm -M versatilepb -kernel vmlinuz-3.2.0-4-versatile -initrd initrd.img-3.2.0-4-versatile -hda debian_wheezy_armel_standard.qcow2 -append "root=/dev/sda1" -usbdevice disk:/dev/disk1s1
```

Now `ssh` into the VM as `root`, the password is `root`. Find, mount, and `chroot` the USB drive. `<USB>` being the USB drive.

```
$ fdisk -l
$ mkdir /tmp/pogo
$ mount /dev/<USB> /tmp/pogo
$ chroot /tmp/pogo /bin/bash
```

Maybe you need to fix `sshd_config`, like I did.

```
$ vim /etc/ssh/sshd_config
```

Unmount the drive and shutdown the VM. The USB drive will then return to the host machine, where we can safely eject it using Disk Utility.

```
$ umount /tmp/pogo
$ shutdown -h now
```

Go plug that SATA drive back into your PogoPlug and pray to Stallman that everything works.

Big thanks to [aurel32](https://people.debian.org/~aurel32/) for hosting!

[PogoPlugv4]: https://colbyolson.com/pogoplugv4
[Homebrew]: http://brew.sh
[USB to SATA Adapter]: http://www.amazon.com/s/ref=nb_sb_noss?url=search-alias%3Dcomputers&field-keywords=usb+to+sata
[ARM]: https://en.wikipedia.org/wiki/ARM_architecture
[QEMU]: http://wiki.qemu.org/Main_Page
