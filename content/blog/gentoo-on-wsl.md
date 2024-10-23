---
title: Gentoo on Windows with WSL2
author: throttlemeister
date: 2021-01-19T15:55:01+00:00
categories:
  - Computer
  - WSL
tags:
  - gentoo
  - linux
  - windows
  - wsl
  - wsl2

---

# Introduction

WSL 2 on Windows 10 introduced the ability to run a native linux kernel on your computer while using Windows 10 as your main operating system. Instead of emulating a linux kernel, like WSL 1 does, WSL 2 uses a hypervisor to run linux in parallel with Windows.  
To be able to run WSL 2 on Windows 10, installation of Windows 10 feature update 2004 or newer is required.

# Enabling WSL2 support

Open an administrator PowerShell by pressing Windows + X, then A or select PowerShell (Run as Administrator) from the start menu. Execute the following commands to enable the hypervisor to start on system boot.

    Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
    Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform

Download the latest stage 3 build from the gentoo website (<https://gentoo.org/downloads/>). Select the profile which suits your needs. In my case I decided to use the most up-to-date, non-hardened, no-multilib, amd64 stage3 build.

Quick-Link to index: [current-stage3-amd64-nomultilib](https://gentoo.osuosl.org/releases/amd64/autobuilds/current-stage3-amd64-nomultilib/)

non-hardened: This is only a development system, so no hardening is needed.  
no-multilib: Who needs x86 support anyway when compiling from source?  
amd64: Because an up-to-date Windows 10 should be running on an up-to-date amd64 plaform.  
Convert the tar.xz archive into an uncompressed tar archive by opening it with 7-Zip and selecting the extract option from the menu. Or use a Linux distro on WSL already installed. Do not extract the linux filesystem packed in the tar archive.

Open an administrator command promt / PowerShell by pressing Windows + X, then A or select PowerShell (Run as Administrator) from the start menu. Run the following command to import the userspace image into your WSL configuration. Be sure that you provide full paths and didn't forget the version argument. The destination path can be adjusted to your liking and is preferably on an SSD with at least 25 GB of free space. The import command will create a single virtual disk image (ext4.vhdx) in the destination folder.

    wsl.exe --import "Gentoo" "D:\path\to\your\installation\directory" "C:\path\to\your\folder\stage3-amd64-nomultilib-20191113T214501Z.tar" --version 2

If you forgot the version switch, the result of the command above is an extracted linux filesystem in the specified Windows folder. Remove the distribution with wsl unregister and restart the import operation.

# WSL setup in Gentoo

As with every new piece of software, the configuration needs some changes on the WSL / Gentoo side to run smoothly. Run the following commands on the Gentoo instance to fix problems with a non-functional nameserver configuration and file system attributes on Windows drives.

    #!/bin/bash
    set -e -x
    
    # Delete auto-generated files
    rm /etc/resolv.conf || true
    rm /etc/wsl.conf || true
    
    # Enable changing /etc/resolv.conf
    # Enable extended attributes on Windows drives
    cat EOF  /etc/wsl.conf
    [network]
    generateResolvConf = false
    
    [automount]
    enabled = true
    options = "metadata"
    mountFsTab = false
    EOF
    
    # Use google nameservers for DNS resolution
    cat EOF /etc/resolv.conf
    nameserver 8.8.8.8
    nameserver 8.8.4.4
    EOF

Create a .wslconfig configuration file for WSL in the Windows user directory. This is necessary to set a maximum size limit of the RAM which WSL may consume. Sometimes the linux kernel uses a portion of the free memory as cache and therefore will start to eat away all RAM of the host system. This can be mitigated by setting the memory configuration option. Replace YOURUSERNAME with your Windows username in the script below.

    #!/bin/bash
    set -e -x

    cat EOF  /mnt/c/Users/YOURUSERNAME/.wslconf
    [wsl2]
    #kernel=
    memory=8GB
    #processors=
    #swap=
    #swapFile=
    localhostForwarding=true
    EOF

Now close all instances of Gentoo / WSL and stop the virtual machine by opening the Command Prompt (Windows-Key + X, then A) and issuing the command wsl shutdown.

# Gentoo setup in WSL

Run all following commands in the Gentoo instance.

For gentoo to work correctly under the hypervisor some feature flags have to be disabled. Open the file /etc/portage/make.conf and adjust the following settings. Below is an example configuration used for my system.

 Add FEATURES=-ipc-sandbox -pid-sandbox -mount-sandbox -network-sandbox to disbable the non-functional sandboxing features  
 Adjust COMMON_FLAGS to match your PC architecture.  
 Adjust USE to your needs  
 Ajdust MAKEOPTS to the number of CPU cores (+1) to make the compilation faster  
 The other options are all optional

    # No GUI (-X -gtk), only english error messages (-nls)
    USE="-X -gtk -nls"

    # Enable python 3.7 and set 3.6 as default
    PYTHON_TARGETS="python3_6 python3_7"
    PYTHON_SINGLE_TARGET="python3_6"

    # Define targets for QEMU
    QEMU_SOFTMMU_TARGETS="aarch64 arm i386 riscv32 riscv64 x86_64"
    QEMU_USER_TARGETS="aarch64 arm i386 riscv32 riscv64 x86_64"

    # No hardware videocard support
    VIDEO_CARDS="dummy"

    # Disable non-functional sandboxing features
    FEATURES="-ipc-sandbox -pid-sandbox -mount-sandbox -network-sandbox"

    # Always ask when managing packages, always consider deep dependencies (slow) EMERGE_DEFAULT_OPTS="--ask --complete-graph"

    # Enable optimizations for the used CPU
    COMMON_FLAGS="-march=haswell -O2 -pipe"
    CHOST="x86_64-pc-linux-gnu"
    CFLAGS="NULL"
    CXXFLAGS="NULL"
    FCFLAGS="NULL"
    FFLAGS="NULL"
    MAKEOPTS="-j5"

    # NOTE: This stage was built with the bindist Use flag enabled
    PORTDIR="/var/db/repos/gentoo"
    DISTDIR="/var/cache/distfiles"
    PKGDIR="/var/cache/binpkgs"

    # This sets the language of build output to English.
    # Please keep this setting intact when reporting bugs.
    LC_MESSAGES=C

# Final installation steps

To finish the Gentoo installation a new snapshot of the ebuild repository should be downloaded. A recompilation of the compiler ensures that GCC is on the most recent stable version. After updating GCC a recompilation of all programs / libraries ensures that the set optimizations take effect.

    #!/bin/bash
    set -e -x

    # Download a snapshot of all official ebuilds
    emerge-webrsync

    # Upgrade the compiler and the required libtool library
    emerge --oneshot --deep sys-devel/gcc
    emerge --oneshot --usepkg=n sys-devel/libtool

    # Update all packages with the newly built compiler
    # This will take a long time, ~1-5 hours
    emerge --oneshot --emptytree --deep @world
    emerge --oneshot --deep @preserved-rebuild
    emerge --ask --depclean

# Enabling overlays for portage

Eselect provides an easy integration of overlays into portage. The main portage respository should already be configured properly, so only a simple installation of eselect is necessary. For more information see the official wiki. Run the command shown below to install the repository module for eselect.

    #!/bin/bash

    # Install portage overlays
    emerge --ask app-eselect/eselect-repository

The configuration for the plugin is located in the /etc/eselect/repository.conf file. The default path of the repository index is specified by the REPOS_CONF option which points to /etc/portage/repos.conf by default. Make sure that this directory exists or create it with the following command.

    mkdir -p /etc/portage/repos.conf

A file named gentoo.conf should be located in this directory (/etc/portage/repos.conf) which holds the configuration for the main gentoo repository. Below is an example default configuration file which was created on my system.

    [DEFAULT]
    main-repo = gentoo
    
    [gentoo]
    location = /var/db/repos/gentoo
    sync-type = rsync
    sync-uri = rsync://rsync.de.gentoo.org/gentoo-portage/
    auto-sync = yes
    sync-rsync-verify-jobs = 1
    sync-rsync-verify-metamanifest = yes
    sync-rsync-verify-max-age = 24
    sync-openpgp-key-path = /usr/share/openpgp-keys/gentoo-release.asc
    sync-openpgp-keyserver = hkps://keys.gentoo.org
    sync-openpgp-key-refresh-retry-count = 40
    sync-openpgp-key-refresh-retry-overall-timeout = 1200
    sync-openpgp-key-refresh-retry-delay-exp-base = 2
    sync-openpgp-key-refresh-retry-delay-max = 60
    sync-openpgp-key-refresh-retry-delay-mult = 4
    sync-webrsync-verify-signature = yes

The synchronization of emerge can now be done via the command below. This might take some time, depending on the internet speed and if the repository was synchronized before.

    emerge sync

# Using Git for portage sync

An alternative method to sync the portage repository is to use git. For this the git package has to be installed on the system as shown below.

    #!/bin/bash

    # Install the git version control system
    emerge --ask dev-vcs/git

Edit the**sync-type**and**sync-uri**in the portage configuration file under /etc/portage/repos.conf/gentoo.conf as shown below.

    [DEFAULT]
    main-repo = gentoo

    [gentoo]
    location = /var/db/repos/gentoo
    #sync-type = rsync
    #sync-uri = rsync://rsync.de.gentoo.org/gentoo-portage/
    sync-type = git
    sync-uri = https://github.com/gentoo-mirror/gentoo.git
    auto-sync = yes
    sync-rsync-verify-jobs = 1
    sync-rsync-verify-metamanifest = yes
    sync-rsync-verify-max-age = 24
    sync-openpgp-key-path = /usr/share/openpgp-keys/gentoo-release.asc
    sync-openpgp-keyserver = hkps://keys.gentoo.org
    sync-openpgp-key-refresh-retry-count = 40
    sync-openpgp-key-refresh-retry-overall-timeout = 1200
    sync-openpgp-key-refresh-retry-delay-exp-base = 2
    sync-openpgp-key-refresh-retry-delay-max = 60
    sync-openpgp-key-refresh-retry-delay-mult = 4
    sync-webrsync-verify-signature = yes

Portage will now complain when synchronizing with git for the first time that the repository folder is not empty and that the folder can not be used to clone the git repository into it. This can be fixed by deleting the old rsync-managed repository which is located under /var/db/repos/gentoo when using the default configuration. Use the command below to do this.

    #!/bin/bash
    
    # Remove old and generate new repository
    rm -r /var/db/repos/gentoo
    emerge --sync

    # Sync twice to test synchronization speed
    emerge --sync</code></pre>

    **Final bits and bobs and clean-up** Now that we have Gentoo installed and working in WSL2, we should do some cleaning up and do some things to make our lives easier.

    First, it would a good idea right now to create and export of our installed Gentoo WSL distribution.

        wsl.exe --export Gentoo gentoo.tar

Store this file somewhere safe. You can use this later to import the Gentoo again if needed, or use it on another computer without having to go through all the steps above.

Now would also be a good time to add a user to Gentoo that you would normally use, instead of using root. If you are installing Gentoo and have come this far, I probably don't need to explain how to create a user on Linux. So I am not going to.

After you have created the user, note the user ID. Normally, this would be 1000.

To set the default user for Gentoo to use, we cannot use the normal procedure as used for distributions installed from the Windows store. Instead, we need to open the registry editor and navigate to:

    HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Lxss</code></pre>

You will see a list of folders with seemingly random names. One of these is for Gentoo, probably the bottom one. Find the one for Gentoo, and edit the key named DefaultUid and set the value to (decimal) 1000 or the number from your user ID if it is something else.

**WARNING**: if you do this before you have created your username on Gentoo, the distribution will be unusable, however you can fix it by changing the DefaultUid back to zero. Changes are in effect immediately; no need to reboot after changing this value.

**PRO TIP**: If you want to be able to change parameters using the .exe of the distribution as you can when you are using for instance Ubuntu or Debian from the store, grab the .exe from one of these store distro's and put it in the directory where you installed Gentoo. Then rename the file to the name of the distribution as you have added it to WSL. You can now start your distribution using that .exe and for instance set the default user the normal way.
