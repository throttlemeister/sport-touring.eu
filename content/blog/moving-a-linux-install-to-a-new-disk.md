---
title: Moving a Linux install to a new disk
author: throttlemeister
date: 2021-11-08T19:54:03+00:00
categories:
  - Computer
  - Linux
tags:
  - linux

---
Recently I had to move my Linux install from one drive to another, as I was experiencing some issues with a WD SN550 nvme drive causing some short random freezes of the GUI with IO intensive tasks. Since I also have a Samsung nvme drive installed, I decided to see if the problem persists on the other drive.

But. Having a fully configured and customized Linux install is a pain in the behind to redo. I did not want to clone, because I made a mistake during install the previous time and it installed in legacy MBR mode, so I wanted to do a proper install using UEFI mode. But preferably not having to re-do all the setup and customizations.

And I didn't have to. Apt to the rescue.

    $ apt-mark showauto  pkgs_auto.lst
    $ apt-mark showmanual pkgs_manual.lst

This will generate a list of .deb packages installed on the system. The first one with all the automatically installed packages, the second one all the manually installed packages from the commandline.

I also made a backup of `/etc/apt/sources.list.d` and `/etc/apt/trusted.gpg.d`. The first directory contains all the repositories I had in use on the original install, and the second directory holds all the GPG keys that go with these repositories. Important!

First install the system on the new drive, and make sure all updates are installed. You don't need to install or setup anything but the base system. Now you can continue with the back ups and files you created earlier.

After I moved the two directories above to their respective place on the new install, and of course doing an sudo apt update && sudo apt upgrade to make sure all packages are still up to date, I loaded up the list of packages I created earlier.

    $ sudo apt install $(cat pkgs_auto.lst)
    $ sudo apt install $(cat pkgs_manual.lst)

This will create some errors due to packages that cannot be installed like this, or packages that were installed from a .deb file and aren't located in any repository. Clean up those entries, and try again and let it run.

When it is finished, copy your /home/<user> from your old drive to your new drive and when you reboot and log back in as your user, everything should be just as it was before.

Success!
