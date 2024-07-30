+++
title = 'Very Useful Utility: Plop Boot Manager'
date = 2011-05-27T10:00:02-04:00
draft = false
tags = ['utilities']
+++

A couple years back I stumbled across a very handy utility written by Elmar Hanlhofer called [Plop Boot Manager](http://www.plop.at/en/bootmanager.html). As the name suggests, it is actually a very small boot loader that is packed with uncommon features. Just a couple key features: [^1]

- USB boot without BIOS support (UHCI, OHCI and EHCI)
- Start of the boot manager from harddisk, floppy, USB, CD, DVD
- Starting from Windows boot menu
- Starting from LILO, GRUB, Syslinux, Isolinux, Pxelinux (network)
- It can be integrated into the BIOS as a PCI option ROM
- [Full list of features…](http://www.plop.at/en/bootmanager.html#features)

Right about now you may be asking yourself “What’s the big deal? Is there really any reason to write another boot loader?” First, there is always some reason or another to write a new version of a piece of software. Secondly, it is true that there are a lot of boot loaders out there already, but what makes this one stand out?

Picture this scenario, a friend of yours gives you an old Pentium 4 laptop. This laptop works fine but has a worn out copy of Windows XP on it. It is equipped with a plain CD-ROM, two USB 1.1 ports, a PCMCIA slot, a floppy and an ethernet port. You think to yourself, “Hey! I’ll just throw a copy of Linux on there and that will be a fun toy to experiment with.” You go online to the Linux distribution of your choice and then spend a few hours downloading all of the necessary ISO files. Once everything is downloaded, you burn them to disk. You’re ready to start installing your shiny new copy of linux. You power on the laptop, pop in the disc, but nothing happens.

After unsuccessfully trying to get the laptop to boot to the disc, you determine that the CD-ROM has failed and you are pretty much SOL. So then you download the USB version, put it on a USB drive and try to boot. Not very surprisingly, that fails to boot as well. Since the laptop is 10 years old or more. USB booting isn’t really possible. For the really determined (and nerdy), you could try setting up a TFTP server and boot using PXE. This is something that is usually doable, but also a pain in the ass. Since the kernel size of a modern Linux distribution is larger than a floppy, I guess you are out of options then, right? Wrong.. This is where Plop shines.

Plop Boot Manager is a very handy tool to keep around. Plop is capable of being loaded on just about any media, and booting to just about any source. In a situation like what I mentioned above, Plop could rescue you. First you will have to find you an old 3.5″ floppy, [download a copy of Plop](http://www.plop.at/en/bootmanager.html#download) and install it on the floppy. I will not go into how to do this, but you can find the directions [here](http://www.plop.at/en/bootmanager.html#noinstall). Once you have installed it on the floppy, you can boot to your floppy and then choose to boot to that USB stick that you made earlier (you didn’t erase the USB drive, did you??). Now that you have done this, the Linux Kernel takes over and it should be clear sailing from here.

There are only two drawbacks that I have seen when working with Plop. The first one being that it is NOT open source, however it is free to use for personal or non-commercial use. The second is that there is currently no support for booting to a USB CD-ROM. I know that he has been working on it, but it appears that it is much more difficult to write a BIOS level driver for a USB optical drive. (Correction: after speaking with Elmar I have been informed that it is a lack of necessary time to develop the USB Optical driver. He has informed me that USB Optical drive support is going to be included into Plop Boot Manager 5.1.) With these things said, I can’t complain over two small drawbacks when a tool is this well written.

[^1]: This partial list of features was taken from the Plop home page, to see all features, please check out the home page