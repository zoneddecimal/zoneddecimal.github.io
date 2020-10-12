---
layout: post
title: GRUB bootloader corruption
category: Tech
---

## GRUB error
While upgrading my version of Virtualbox guest additions (6.1.12 to 6.1.14) on my Debian guest, the VBoxLinuxAdditions.run script somehow silently failed and apparently corrupted my GRUB bootloader.  Sigh.  Starting the virtual machine gave me this cryptic error.

```
error: symbol 'grub_calloc' not found
```

After some googling I attempted to try manually loading the `normal` module via GRUB, just to see what happens.  Indeed, the same error...

![image](/images/GRUB_bootloader_corruption/2020-10-11_180636.png)

## Debian netinst
At this point it seems the simplest way of fixing it would be to reinstall the grub bootloader.  Luckily, the debian installation images comes with a rescue mode to do exactly that.

I downloaded the latest Debian netinst image from [here](https://www.debian.org/distrib/netinst).

What we need to do is have our virtual machine boot with that image rather than our usual virtual hard disk.  To do this, follow these instructions on the host system.

1. Start virtualbox
1. In the Settings for the affected virtual machine, go to Storage and add a new optical drive under the IDE controller.  The source of the optical drive should be the previously downloaded Debian netinst ISO file.
1. Start the virtual machine, and immediately hit F12 to select your boot device.  Select `c` for cd-rom.
1. The debian installation will start up.  Select Graphical Rescue Mode.
1. Follow the next installation steps (language, location, keyboard layout, hostname, domain name).  These settings do not matter since we are not actually installing Debian.
1. On the next screen you will be prompted to select the partition to use as a root filesystem.  In my case it was /dev/sda1.  If you choose wrongly it will say it cannot find a root filesystem on the device.
1. Finally, select Reinstall GRUB bootloader.

![image](/images/GRUB_bootloader_corruption/2020-10-11_190440.png)

On the next screen enter the name of the device on which to install GRUB.  In my case it was `/dev/sda`.

![image](/images/GRUB_bootloader_corruption/2020-10-11_190527.png)

After it installs GRUB, choose Reboot the system.

Success!  GRUB has been reinstalled!

