---
title: 'As short as possible: application containers, system containers, virtualization'
author: throttlemeister
date: 2021-02-13T10:09:41+00:00
categories:
  - Computer
  - containers
  - virtualization
tags:
  - containers
  - kvm
  - virtualization

---
Sometimes, there is a bit of confusion as to what the difference is between application containers (Docker, Podman, K8S, OpenShift), system containers (LXC/LXD) and virtualization (KVM, Vmware, Hyper-V, Xen) and when you should use them. I will try to explain the differences as short as possible.

# Application containers

Application containers are created with the most minimal environment to run a specific application. This includes the OS and all dependencies for that application. Every tool or program normally present in the OS that is not needed to run the application is typically left out so that the container image is a small as possible and performs as fast as possible.  
  
Examples of application containers are Docker, Podman, Kubernetes and OpenShift.

# System containers

System containers are typically as the minimal environment needed to run a specific operating system. All the basic tools are present, including the package manager so you can set up the system as you want with all the tools you need. A system container is like a virtual machine, but without the hardware virtualization layer. A system container uses and identifies the host hardware and runs the same kernel as the host. This means it is much lighter on resources than full blown virtualization, but also that in certain cases it can be incompatible with certain software.  
  
Examples of system containers are LXC, LXD

# Virtualization

Virtualization is software that emulates the hardware so that more than one virtual machines can be installed on the same physical hardware at the same time. As the full hardware layer needs to be simulated, it has the most overhead of these options but it also is the most compatible with all software.  
  
There are two types of virtualization:  
  
1. Type 1 hypervisors: virtualization is done by the kernel, providing low-level access to the physical hardware for the virtualization software for increased performance.  
Examples of Type 1 hypervisors: KVM, Vmware ESXi, Hyper-V Server  
2. Type 2 hypervisors: virtualization is done by an application installed on an operating system. Access to hardware must be done through the OS and no direct access is possible. IT provides convenience over performance, as it can run on any sytem.  
Examples of Type 2 hypervisors: Vmware Workstation, VirtualBox, Hyper-V manager

