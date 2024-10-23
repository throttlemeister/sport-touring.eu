---
title: No more Windows. FreeBSD instead.
author: throttlemeister
date: 2023-10-10T17:00:00+00:00
categories:
  - BSD
  - Computer

---
No, I did not stop running OpenSUSE Tumbleweed. That's still my daily driver. I did however wipe my Windows drive and installed FreeBSD instead. I have used FreeBSD quite a lot way back when, and since I hadn't actually booted into Windows for months, I just decided to install FreeBSD on that drive so I could play with it again after I saw KDE is actively supporting FreeBSD for the desktop.

Installing FreeBSD is actually pretty straight forward. It uses an ncurses installer that was common on BSD and Linux alike 25 years ago, but today it's basically only seen on FreeBSD and Slackware Linux. You can however use that GUI over SSH, so pretty convenient for server installs.

After answering a few simple questions like locale, disk and services you're essentially done and prompted to reboot. Then comes the scare for the modern Linux user: FreeBSD boots into a command prompt. No GUI. Not installed.

Oops. 🙂

Seriously, not that complicated either. It's just that you have consciously decide you want it, and what you want. That's not a bad thing. Arch Linux users will understand.

To use KDE on FreeBSD, we need to install some things and configure a few others. To install:

    pkg install --quiet --yes kde5 plasma5-sddm-kcm sddm xorg

If you are running an nvidia graphics card, do yourself a favor and also install:

    pkg install --quiet --yes nvidia-xconfig

Then run the following commands to configure (as root):

    nvidia-xconfig
    sysrc dbus_enable="YES" && service dbus start
    sysrc sddm_enable="YES" && service sddm start

The SDDM login screen should start and you should be able to login to KDE. Do make sure you choose X11 for session, not Wayland as Wayland is extremely buggy on FreeBSD at best. 

If no GUI, try to reboot first and if that doesn't work, check if you have an xorg.conf under /usr/local/etc/X11 as this should have been generated by the nvidia-xconfig utility, but without xorg config nothing will happen.