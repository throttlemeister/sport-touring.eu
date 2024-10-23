---
title: The end to distro-hopping?
author: throttlemeister
type: post
date: 2023-08-04T21:33:15+00:00
url: /the-end-to-distro-hopping/
categories:
  - Computer
  - Linux
  - OpenSUSE

---
Since I have moved to running Linux basically full time, I have ran Pop!OS, Fedora, Nobara, back to Fedora KDE Spin and now OpenSUSE Tumbleweed. All of them are good distributions and none have any real deal-breakers for me not to want to use them.

The fact that I really, really like Tumbleweed was not a given. Quite the contrary. OpenSUSE does it's own thing, so it is a bit of a learning curve if you are used to Debian-based or RedHat-based distributions. It is one of the oldest distributions still operating today, which puts it in the same company as Debian and Slackware. The latter actually being the initial origins for (then) SUSE Linux.

But I digress. My experience with SUSE in the past (=20 odd years ago) was less than favorable to the point that I abandoned it and never really looked at it again until now. Installation was painless and it let you choose your desired desktop environment as part of the installation procedure. I run KDE, which is the first option and let it do its thing.

After install I was back in business. Mind you: I have /home on a seperate disk, so all I have to do when reinstalling or using a new distribution is to make sure that drive is mounted on /home and I have all my (program) settings back and my desktop just looks the same as before. Saves me time to set everything up the way I want again and no data loss when doing a reinstall.

Some observations after installing OpenSUSE Tumbleweed:

- Using OpenSUSE means using BTRFS right: proper subvolumes upon install, snapper configured out of the box to create snapshots when installing software and updates and grub configured to be able to boot from previously created snapshots so you can rollback if something happens to go wrong. No more worried about effing up your system
- YaST2 is a very useful tool. It let me configure and add my iscsi targets in 2 minutes tops and it connects automatically on boot again, whereas most other distributions make that a painful, manual task that requires workarounds to mount your drives after a reboot. I don't really need a GUI tool to configure my systems, but this tool lets you configure all aspects of the system and sometimes it is just faster doing it once in the GUI than to do it manually.
  **Note**: the original YaST is the reason I quite SUSE years ago and never came back. Different story.
- It has a huge repository of programs, including non-FOSS if you add the packman repos. And if you install opi (openSUSE Package Installer) you have the entire OBS (OpenBuild System) at your finger tips, including user repositories. Basically you have the OpenSUSE equivalent of Arch's AUR.
- Installation of proprietary drivers and codecs is easy and painless, as with most distributions. That said, I think OpenSUSE is the only distro where the Nvidia drivers come straight from a repo at Nvidia.
- It's (Tumbleweed) a rolling distribution, so always the latest software but it is better tested than something like Arch. This means less issues are likely to arise.

Some inconveniences I ran into:

- Both my laptop and desktop were unable to load Wayland and had to use Xorg. This is not a big thing. This was caused by a SDDM bug where the SHELL variable got messed up if you use the fish shell as you login shell. This has been fixed by the KDE team in '21/'22, but the fix was never backported to OpenSUSE by the OpenSUSE team. Workaround was to change the login shell to standard bash, and just tell the terminal app to start with the fish shell. Now I can run Wayland on my laptop, while still using Xorg on my desktop as it work just works better with Nvidia graphics. But it will run Wayland if I want to.

Other things I did after installing Tumbleweed are not OpenSUSE specific things, but more generic Linux optimizations I have made.

- I used tuned with tuned-adm to select and set an appropriate tuning profile for my systems.
- I have set up a couple scripts for my laptop to switch profiles and CPU modes depending on if the laptop is running on AC power, or battery.

When running on AC power:

    #!/bin/bash
    #
    # Set CPU governor to powersave
    if [ -e /usr/bin/cpupower ]; then
        sudo cpupower frequency-set --governor performance
        sudo tuned-adm profile latency-performance
    else
        echo performance | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
        sudo tuned-adm profile latency-performance
    fi
    # EOF

When running on battery power:

    #!/bin/bash
    #
    # Set CPU governor to powersave
    if &#91; -e /usr/bin/cpupower ]; then
        sudo cpupower frequency-set --governor powersave
        sudo tuned-adm profile powersave
    else
        echo powersave | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
        sudo tuned-adm profile powersave
    fi
    # EOF

These are set in KDE settings under power management, see screenshot:
![Screenshot](/images/SS_LOKI_Energy-Saving-—-System-Settings_20230804_01-1024x667.png)

- On the desktop, I just set the tuned profile to latency-performance using tuned-adm. This is a profile that aims to prioritize latency over throughput, which is great for desktop use with a GUI.
- I also set some sysctl parameters as outlined in another post.

Like I said, these are not Tumbleweed or OpenSUSE specific optimizations or even necessary, but it is how I prefer to set up my systems. I feel it helps me to squeeze out the most performance out of my systems, and in the case of the laptop, preserve battery life when needed.

When all is said and done, I really like OpenSUSE Tumbleweed. I like the bootable snapshots. I like having the latest versions of different software without having to wait for a next release. I like the tools provided to admin the system. It's a good'un. I think I'll keep it. Then again, I said that of Fedora. And Nobara. But I do think it is true this time.
