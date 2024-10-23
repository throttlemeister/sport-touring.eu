---
title: Optimizing Linux for desktop performance
author: throttlemeister
date: 2021-11-17T19:31:00+00:00
categories:
  - Computer
  - Linux
tags:
  - linux

---
My daily driver is currently Pop!_OS, which is a desktop Linux distribution. It's a very nice distribution, really good with Nvidia hardware (which isn't a given on Linux) and, to me, a Gnome look that is very close to what I want so my GUI changes are minimal.

What's less, and that is a more generic Linux problem, is that particularly the Linux kernel is optimized for server use and not desktop. It prioritizes throughput over latency, which is great for raw performance but less if you expect a smooth, fast GUI.

We can fix that.

# Kernel

This first one is optional and controversial. Many will say a custom kernel is not needed and does not add anything. On my computer however, using the [Xanmod](https://xanmod.org/) kernel does make the GUI significantly faster and smoother. Installation instruction are on their page.

Second, we want to pass two boot parameters to the kernel when booting. If using systemd-boot, like Pop! does, open the corresponding file under `/boot/efi/loader/entries` and add:

    nvme_core.default_ps_max_latency_us=0 pcie_aspm=off

to the line starting with options.

If using grub, add the same to the line `GRUB_CMDLINE_LINU`X_DEFAULT under `/etc/default/grub` and do a update-grub to activate.

# Options

Part two is modifying the sysctl parameters. Under `/etc/sysctl.d` you will find files that set certain parameters on how your system works. Create a new file, and add the following:

    # These are settings from /etc/sysctl.d/ and can be activated by running sysctl --system as root
    # Save this file in that location.
    #
    # These settings set the disk caching for the system
    #
    vm.dirty_bytes = 33554432
    vm.dirty_background_bytes = 8388608
    vm.dirty_writeback_centisecs = 100
    vm.dirty_expire_centisecs = 300
    #
    # We need to either use *_ratio, or we need to use *_bytes. We cannot use both. Currently
    # using _bytes, so disabling _ratio
    #
    # vm.dirty_background_ratio = 10
    # vm.dirty_ratio = 80
    #
    vm.page-cluster = 0
    # Increased to improve random IO performance
    fs.aio-max-nr = 1048576</code></pre>

This will set certain parameters pertaining to disk caching and IO performance. You can activate by running sysctl system as root, or by rebooting your system.

# Disk

To optimize your disks, if you are using SSD, it's worth it to make some changes to your `/etc/fstab`. There's two parts to this:

1. Mount the root filesystem with settings optimized for SSD's
2. Ensure temporary directories are running from memory by mounting them in a tmpfs to limit disk writes and extend the life of your SSD.

For the first one, I mount my root device in `/etc/fstab` like:

    device  /  btrfs  defaults,ssd,noatime  0  0

For the second, add these lines to `/etc/fstab`

    # SSD tweak: temporary directories as tmpfs
    tmpfs   /tmp       tmpfs   defaults,noatime,mode=1777   0 0
    tmpfs   /var/tmp   tmpfs   defaults,noatime,mode=1777   0 0
    tmpfs	/var/log	tmpfs	defaults,noatime,mode=0755	0	0
    tmpfs	/var/spool	tmpfs	defaults,noatime,mode=1777	0	0

>**DISCLAIMER**: Putting anything other than `/tmp` into memory, can produce unpredictable results in specific circumstances. It should be ok on desktop machines and helps to extend the life of your SSD by limiting writes. Do not enable on servers. Actually most of what is on this page may have an adverse effect on server performance.

Activate by rebooting your system. Enjoy a faster, more responsive system.

>**[FEB 8/22 UPDATE]**: Since publishing this article I have moved from Pop!_OS to Fedora. Fedora is a cutting edge distribution, which does not require all of these tune-ups to make a snappy OS out of the box.

Let's elaborate.

1. The kernel parameters mentioned above do **not** need to be updated on Fedora
2. The sysctl.d modifications are **not** required on Fedora, but they are done simply because I have more than plenty memory anyway. Out of the box default settings on Fedora are better than those of Pop!_OS
3. The disk optimizations in /etc/fstab are set by default

# Fedora Update

Fedora uses a different package management system than Pop! OS, which is Ubuntu based. While Debian derivatives like Ubuntu and Pop!\_OS use apt as their package manager, Fedora is RedHat based. RedHat uses rpm files which are managed by yum or dnf (depending on the version of the OS).

By default, dnf is quite slow compared to apt but this is easily fixed by adding some parameters to the configuration file.

    [main]
    gpgcheck=1
    installonly_limit=3
    clean_requirements_on_remove=True
    best=False
    skip_if_unavailable=True
    &lt;strong>max_parallel_downloads=10
    fastestmirror=True

The two bottom bold lines need to be added to /etc/dnf/dnf.conf. The first one increases the number of simultaneous download connections to 20, which increases download speed. The second one looks for the fastest mirror from your location, which ensures you will get the maximum possible download speed. Combined, these make dnf operate as fast or faster than apt on most systems.

Due to the nature of a bleeding edge distribution like Fedora, it can sometimes be tricky to update. Especially the kernel and / or kernel drivers. To avoid such problems, I run updates with the `--exclude=kernel*` flag. In fact, I wrote a function for my Fishshell to get and install updates without kernel, like so:

    function up2date
      sudo dnf upgrade -y --refresh --exclude=kernel\*
    end

And saved it as `up2date.fish` under `$HOME/.config/fish/function`s
