---
title: Any Linux distribution on Windows 10 with WSL2
author: throttlemeister
date: 2021-01-19T15:57:01+00:00
categories: ["computer","WSL"]
tags: ["docker","linux","windows","wsl","wsl2"]
---
**Introduction&nbsp;**&#8211; WSL 2 on Windows 10 introduced the ability to run a native linux kernel on your computer while using Windows 10 as your main operating system. Instead of emulating a Linux kernel, like WSL 1 does, WSL 2 uses a lightweight hypervisor to run linux in parallel with Windows.  
To be able to run WSL 2 on Windows 10, installation of Windows 10 feature update 2004 is required.  
  
An explanation on how to enable WSL2 support can be found on the page detailing how to create a Gentoo instance on WSL2.  
  
**Any Linux distribution on Windows?**&nbsp;&#8211; Yes, you can run any distribution of Linux on WSL quickly and easily, provided that:&nbsp;&nbsp;&nbsp;&nbsp;  
&nbsp; &nbsp;1)&nbsp;[a Docker image for that distribution is available.][1]  
&nbsp; &nbsp;2)&nbsp;[you install your distribution as normal in a virtual machine first][2]

**For the Docker method**

<pre class="wp-block-code"><code>docker pull &lt;image name for your Linux distro&gt;
docker create --name &lt;distro&gt; &lt;image name for your distro&gt;
docker export -o &lt;distro&gt;.tar &lt;distro&gt;

wsl --import "&lt;distro&gt;" "&lt;location for your wsl distro&gt;" "PATH/TO/&lt;distro&gt;.tar" --version 2</code></pre>

**How does it work**&nbsp;&#8211; The first line, the docker pull, downloads the docker image to your computer. This can be any image, as long as it is Linux based, but since you are trying to get a specific Linux distribution for use in WSL, the assumption is it is an image for a Linux distro.

The second line, the docker create, creates the docker container. It doesn&#8217;t start it, it just creates it so you don&#8217;t have to worry about containers all of a sudden taking up resources/

The third line, the docker export, dumps the container into a tar archive. This is the file we need for WSL and contains the entire Linux distribution. At this point, you can throw away your docker container if you want as we do not need it anymore.

Lastly, we are importing the tar archive we created from the docker container into WSL. We can now boot our newly created Linux distro of our choice in WSL and use it, modify it and work it like we want to.

_**NOTE**: This process works in reverse as well! If you want to create and use a docker image from any WSL2 instance you have created you can simply export the WSL distro to a tar archive and import that into Docker and fire it up!_

**For the virtual machine method**  
After you have installed the virtual machine, log into the virtual machine and issue the following commands.

_**Note:**&nbsp;this also works for physical Linux machine or dual boot system where you want to copy the Linux system to WSL!_

<pre class="wp-block-code"><code>$ sudo su -
# cd /
# tar -cpzf backup.tar.gz --exclude=/backup.tar.gz --exclude=/proc --exclude=/tmp --exclude=/mnt --exclude=/dev --exclude=/sys /</code></pre>

This will create a tar archive of the entire system. This can get quite big, especially if you are doing this from a previously installed system and not a clean, base install so make sure you have enough space to save it.

When the tar file has been created, copy it to your Windows machine somehow and issue the following command:

<pre class="wp-block-code"><code>wsl --import "&lt;Your_Distro_Name&gt;" "&lt;Location_to_store_your_Distro&gt;" "PATH/TO/&lt;archive.tar&gt;" --version 2</code></pre>

This will create the WSL instance by side-loading it and you can start it by issueing wsl.exe -d <Your\_Distro\_Name> or by opening Windows Terminal; it should already be listed in the dropdown menu as one of the options.

 [1]: https://www.sport-touring.eu/linuxwsl.html#content10-1r
 [2]: https://www.sport-touring.eu/linuxwsl.html#content10-2h
