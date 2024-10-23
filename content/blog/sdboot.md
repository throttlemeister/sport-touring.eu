---
title: systemd-boot on OpenSUSE
description: Switch from GRUB2 to systemd-boot on OpenSUSE
date: "2024-01-18T23:01:17+02:00"
publishDate: "2024-01-18T23:01:17+02:00"
---
I am weird sometimes, I know. Ever since I used Pop!OS, which uses systemd-boot as default boot manager, I have fallen in love with it. The simplicity and speed of systemd-boot versus the complexity and bulk of GRUB2 just won me over.

Mind you, GRUB2 is very good at what it does. It’s just that I don’t need all it does. All I need is to boot my machine, nothing more, nothing less. My laptop, on which I am typing this, only has OpenSUSE installed. No dual boot, nothing. My desktop boots OpenSUSE and FreeBSD at the moment, but they do so from seperate disks with their own bootloader. Nice and simple.

Neither need the features of GRUB2.

That said, changing the bootloader is tricky. It’d be nice if OpenSUSE would let you pick systemd-boot at install, so you don’t have to muck with it. However, since end of September, OpenSUSE officially supports systemd-boot.

So, I just had to have it, starting with my laptop.

## HOW-TO

**Note**: I do not use secureboot. Do not blindly follow this tutorial if you are using secure boot: it will break your system. You can do it, but it requires more steps then I will outline here.

First, check if /usr/lib/bootloader/systemd-boot exists. If it does, you’re good.

Remove all OpenSUSE entries from the EFI boot menu, and all others you do not need:

    # efibootmgr --delete --label opensuse-secureboot
Do this for each entry you do not need or want. You can see what entries there are by just typinge efibootmgr without any options.

Next we want to update the configuration, so it knows we are going to use systemd-boot.

    # vi /etc/sysconfig/bootloader 
Change from (probably) LOADER_TYPE="grub2-efi" to LOADER_TYPE="systemd-boot"

Install the systemd-boot utils to add support for snapshots. We are running OpenSUSE, we do want support for our automatic btrfs snapper snapshots.

    # zypper in sdbootutil-snapper sdbootutil-rpm-scriptlets
If zypper complains that it needs to remove GRUB2 to do this, confirm by choosing the solution that will do so. It will both add all the files required for systemd-boot to function and remove all grub2 related files that you will not be using anymore.

Then we want to install our kernels, including snapshots so we can actually boot as we do not have a bootmanager installed right now!

    # sdbootutil install
    # sdbootutil add-all-kernels
Now we’re done. Systemd-boot is installed, GRUB2 is removed and we can reboot our system and enjoy a fast and less bloated setup. If you want to get to the boot menu to boot from a snapshot, just hold the spacebar while booting and it will pop up.

## Updates

Since this article has been written, systemd-boot has been officially added to the boot options of OpenSUSE Tumbleweed and it is now available during install on an UEFI system.
