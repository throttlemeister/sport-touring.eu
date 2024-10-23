---
title: "Transactional updates on OpenSUSE Tumbleweed"
date: 2024-03-23T13:52:22+02:00
image: "img/tuxlogo.png"
draft: false
weight: 100
author: "throttlemeister"
---
It’s possible to use transactional or atomic updates on OpenSUSE Tumbleweed and leveraging the snapshot capabilities of btrfs. It’s actually quite simple; all you need to do is to install the command using zipper.

>Important prerequisite: the filesystem must be btrfs.

    sudo zypper in transactional-update

This will install the command. Next you want to set up some, to prevent some potential issues.

    # Note: this systemd service is required to create RPM dirs in /var as TU 
    # does not mount /var in the new snapshot to maintain atomicity.
    systemctl enable --now create-dirs-from-rpmdb.service

That’s basically it. Now if you run the command:

    sudo transactional-update

The system will create a new snapshot, check for updates, install updates on that snapshot and set that snapshot to boot. If there are no updates, the snapshot is removed again.

**Keep in mind, and this is important: any changes you make to the root filesystem will be gone after you reboot into the new snapshot.**

This is expected behavior, but it does mean if you want or need to make changes you need to make sure you reboot first.

Why would you want this? If you need to apply a big update and you don’t want it to interrupt your work or avoid processes being restarted, you can apply the update on a new snapshot without it interfering or making changes in any way on the running system.

Is it necessary? Absolutely not. It is just another tool in the toolbox. It offers some of the advantages (atomic updates) of an immutable distribution, without having to have an immutable (read-only) filesystem.
