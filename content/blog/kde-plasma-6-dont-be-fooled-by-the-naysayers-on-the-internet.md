---
title: 'KDE Plasma 6: Don’t be fooled by the naysayers on the internet'
author: throttlemeister
date: 2024-05-02T21:34:58+00:00
categories:
  - Computer
  - Desktop
  - KDE
  - Linux
tags:
  - desktop
  - kde
  - linux

---
About a month ago, KDE released their much anticipated next major release of the KDE desktop environment, version 6. The only distributions using it soon after were (obviously) KDE Neon, the KDE showcase / development platform, and the rolling release distributions like Arch and OpenSUSE Tumbleweed. The scheduled release distributions will only ship KDE6 in their next new release.

As it was released, obviously there were some teething problems despite a long and extensive beta test cycle and lots of bigfixing. Some of these were:

- Upgrade being interrupted if done from the GUI, as the session was killed, halting the upgrade process. Easy fix: just restart the upgrade and it will finish. If doing a reboot, because you didn't notice there was a problem, you would end up without a GUI. The best way to upgrade, was to switch to a TTY screen and upgade from the CLI without a GUI running.
- Sometimes, especially with a custom SDDM theme selected, the SDDM screen would fall back to a very ugly UI and an error. This was not critical and setting SDDM to the default breeze theme typicall fixed this.
- If you used custom themes, plasmoids, plasma addons etc, these could potentially be incompatible with KDE6 and refuse to load. This is annoying but to be expected. Just unload and remove or upgrade. Currently most have been upgraded to work with KDE6, provided they are actively maintained.
- The default session was changed from X11 to Wayland for all users, including those with Nvidia hardware. In some cases this could cause some issues with Wayland not loading but X11 would generally still work.

For me personally, the experience has been great despite experiencing the first 3 from the above list. They were annoying when it happened but easily fixed. Since the experience has been smooth, my addons have been upgraded by their maintainers and the Wayland experience has been very smooth. I'm not the biggest fan of Wayland myself (perhaps I will write about that at some point), but it is the future and having to live with it like it is now with KDE6 is not punishment.

KDE Plasma 6 has been a great experience for me. It has sane defaults. The tweaks in certain settings and places are very good  example: the panel configuration, thank you KDE team  now you don't need a degree to decipher what some of the things mean.

It's more of an evolution than a revolution going from KDE5 to KDE6, but what an evolution it is. The best desktop just got better.. And smoother. And more polished.
