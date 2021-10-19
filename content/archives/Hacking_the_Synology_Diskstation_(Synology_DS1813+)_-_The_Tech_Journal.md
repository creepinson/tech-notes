---
title: "Hacking the Synology Diskstation (Synology DS1813+) - The Tech Journal"
slug: "hacking-syno"
---

![Picture of a Raspberry Pi 4 UART connected to a console port on a Synology Disktation 1813+](https://web.archive.org/web/20200920021611im_/https://www.stephenwagner.com/wp-content/uploads/2020/05/20200505_130736-1-150x150.jpg)

As a result of my Synology DS1813+ crashing yet again due to the [Synology Memory issues and Crashing](https://web.archive.org/web/20200920021611/https://www.stephenwagner.com/2019/07/31/synology-memory-issues-and-crashing/) that I’ve been regularly experiencing, I finally decided to try hacking the Synology NAS to run another operating system. Let it also be noted that numerous of my readers are also experiencing these issues as I receive chats and e-mails about this almost on a daily basis.

Under the hood, the DS1813+ is just another x86 computer system. There’s no reason why we shouldn’t be able to hack this to run another Linux distribution or possibly even a BSD variant like FreeNAS.

Ultimately all I want from this is a reliable NAS to perform software RAID and provide an iSCSI target, it would also be kinda cool to see what we can install on it!

I’ve already started preliminary work on this, so keep visiting back as the blog post get’s updated with more and more information on a regular basis. If you feel you can contribute, please don’t hesitate to leave a comment or reach out.

## Current Status

In this section, I’ll be updating it regularly with the current status of my efforts.

Completed:

- Serial console access
- UEFI Shell Access
- GRUB Bootloader Access

See the below sections for information.

## Accessing the DS1813+ system

There’s numerous different approachs we can take to try to gain access to repurpose the Synology Disk Station and install another operating system.

These include:

- Accessing the serial console
- Accessing the BIOS/UEFI and/or bootloader
- Booting from a USB stick or modified HD
- Modifying the USB DOM

The ultimate result we are looking for is to boot our own linux kernel, kick off a Linux or BSD OS installer, or boot from a modified drive that already has Linux installed on it.

### Accessing the serial console

Serial console access to the Synology Diskstation is easily acheived.

I originally found this post which provided me information on the pinouts and the voltage: [http://www.netbsd.org/ports/sandpoint/instSynology.html](https://web.archive.org/web/20200920021611/https://www.netbsd.org/ports/sandpoint/instSynology.html)

While the above post is for older units utilizing architectures other than x86, the pinout information along wiht the voltage is still relevant.

With the Synology unit using 3.3v, you cannot use a normal computer RS-232 interface to connect to it as it runs at 5V. You’ll need to step-down the voltage using a converter or use a RS-232 interface that runs at 3.3v.

In my case, I used a Raspberry Pi 4 and one of the UART ports along with Minicom to access it. The Pi 4 uses 3.3v for UART so it works perfect. You’ll need Rx, Tx, and GRND for the connection to work.

![Picture of a Raspberry Pi 4 with UART connection to ttyS0](https://web.archive.org/web/20200920021611im_/https://www.stephenwagner.com/wp-content/uploads/2020/05/Pi4-UART.jpg)Raspberry Pi 4 UART Connection ttyS0

In my case, I used the ttyS0 UART interface to avoid issues with the Mhz and timing (that’s experienced with using ttyAMA0). To use ttyS0, you’ll need to enable the UART on your Pi boot configs, as well as disable the Raspberry Pi console.

![Picture of a Raspberry Pi 4 UART connected to a Synology DS1813+ serial console connection](https://web.archive.org/web/20200920021611im_/https://www.stephenwagner.com/wp-content/uploads/2020/05/20200505_130736-1024x576.jpg)Raspberry Pi 4 UART connected to Synology Diskstation DS1813+ console port

I used the following command to initialize minicom:

```
minicom -b 115200 -o -D /dev/ttyS0
```

After connecting, I was able to view and interact with the serial console.

### Accessing the BIOS/UEFI and/or bootloader

After gaining serial console access, powering on the Synology DS1813+ results in the following:

```
Intel (R) Granite Well PlatformCopyright (C) 1999-2011 Intel Corporation. All rights reserved.Product Name : GRANITE WELLProcessor : Intel(R) Atom(TM) CPU D2701 @ 2.13GHzCurrent Speed : 2.12 GHzTotal Memory : 4096 MBIntel BLDK Version : Tiano-GraniteWell (Allegro 0.3.7)Miscellaneous InfoMemory Ref Code Version :CDV Ref Code Version : 0.9.0-1P-Unit Firmware Version :P-Unit Location in Flash : 0xFFFB0000P-Unit Location in RAM : 0xDF6F0000No of SATA ports available : 6No of SATA ports enabled : 6Press F10 in 3 seconds to list all boot optionsAny other key to active boot…
```

Unfortunately, I’m unable to press F10 due to terminal emulation issues (it’s also possible they’ve removed this feature to stop someone from doing what I’m doing).

After 10 seconds, the Synology will UEFI boot the GRUB bootloader.

You can browse through the list, edit the entries, as well as run the GRUB command line.

### Booting from a USB stick or modified HD

I attempted to boot numerous different USB sticks containing either OS installers (Linux variants and FreeNAS) with no success. I also tried to boot off an HD connected to one of the SATA ports in the NAS, this was also unsuccesful.

I noticed that out of the 8 SATA connections, ports 1-6 are treated differently (possibly being on a SATA expander) and 7-8 may be accessed by the UEFI, BIOS, or bootloader.

I attempted to chainload a CD image written to a USB stick, however GRUB is not able to see any USB or HDs other than the SATA DOM it’s residing on.

Removing the SATA DOM presents you with a UEFI shell, however you are unable to see, view, or execute any efi files as the shell is unable to read any USB or HD devices other than the SATA DOM.

It appears both the UEFI/BIOS and GRUB have been modified to either not allow access to other bootable devices, or drivers are required which haven’t been incorporated.

In order to execute our own kernel or OS, we may need to modify the SATA DOM.

### Modifying the USB DOM

The onboard USB DOM appears to be the only bootable device that is presented to the UEFI/BIOS.

On a booted system, the DOM appears as the device “/dev/synoboot”.

While logged in to the Synology via SSH, you are unable to mount this device to a mount point. You can however image the device, copy it, and write it to another device on another system.

To image the USB DOM, I ran the following command:

```
dd if=/dev/synoboot of=/volume1/ShareName/synoboot-image
```

I than downloaded the “synoboot-image” image file to another Linux system, wrote it to a USB stick and I was able to mount the partitions.

There are two vfat partitions containing some linux kernels, ramdisks, and the UEFI version of GRUB.

I believe in an effort to move forward, we will need to either modify and incorporate a version of GRUB with extra drivers, or we will need to use the existing version to boot our own kernel and initial ramdisk.

At this point, we’ll need to evaluate how to write to the SATA DOM. There are two options:

- Modify the image we created, and write it back after copying it back to the Synology NAS.
- Find a way to directly mount and access the partitions on the Synology NAS, at this point we are unable due to “access is denied”, however dd reading functions.
- Connect the SATA DOM to the USB headers on another system.

Once we access this SATA DOM, it may be possible to copy the kernels and ramdisks to kick off an OS installer, or better yet install a more feature and driver filled version of GRUB.


