---
title: OpenSUSE Tumbleweed, 6 months in
author: throttlemeister
date: 2024-03-02T23:50:13+00:00
categories:
  - Computer
  - Linux
  - OpenSUSE
tags:
  - linux
  - opensuse

---

Well, here we are, 6 months after installing OpenSUSE Tumbleweed and still no urge to change. The only change I am longing for is the update to KDE Plasma 6, which should be coming soon now that it has officially been released.

Since there are basically no updates since my last post in [December](/posts/opensuse/opensuse-4-months-in/), there isn't much to say. I did however made some changes. A couple of weeks ago, my GTX1080 decided to shit itself such that it triggers the short protection in the power supply so that's out. Bit of a pain since everything is water cooled. I ended up replacing it with a Sapphire R9 390 Nitro which is an AMD based card. Raw speed is about half of that of the GTX1080, but at 50 euros it was cheap and being AMD means no more Nvidia issues on Linux as limited as they were for me. Main issue for me was that I had to manually recreate the initrd whenever a new driver version was released to make it work.

Routing of the watercooling soft tubing looks like crap now, but hey, its working again and at least I can use my machine again.

Changing from Nvidia to AMD was a bit of a pain as I had to remove all Nvidia related packages and I had to manually set the correct kernel parameters to make it work. After that it was painless and working just fine. Drivers are part of the kernel, so no more issues there. Test benchmarks suits show lower fps as to be expected, but also smoother and less jerky graphics. I don't game that much, so I guess I'll be fine for now. Would love me some 6700XT or something.

Tumbleweed keeps on rocking.
