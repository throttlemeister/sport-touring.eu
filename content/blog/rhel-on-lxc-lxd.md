---
title: Did you know... (or, RHEL on LXC/LXD)
author: throttlemeister
date: 2021-02-13T09:40:20+00:00
categories:
  - Computer
tags:
  - containers
  - linux
  - lxc
  - lxd
  - redhat
  - rhel
  - virtualization
  - wsl
  - wsl2

---
1. There are no RedHat Enterprise Linux (RHEL) LXC/LXD container images publicly available?
2. There are LXC/LXD container images available for CentOS and Fedora?
3. You can convert a CentOS install to RHEL using the [Convert2RHEL](https://access.redhat.com/articles/2360841) tool?
4. This also works for a LXC/LXD container?
5. You will need to do this if you want to run RHEL on Proxmox in a LXC container?
6. You can create a tarbal of a running system to import into a vanilla LXC/LXD installation?
7. You will need to create a metafile.tar.gz with a few lines of information about the tarbal to do this?
8. You can also use an export from a Docker container to get the system?
9. And that you can also use an export from a WSL distribution for this?
10. This means you can set up a WSL environment on your Windows box (see other posts here) just the way you want with all the tools you need, and you can export it to run it independently in a LXC/LXD container as a server?
