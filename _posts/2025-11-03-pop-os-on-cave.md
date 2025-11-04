---
layout: post
title: Installing Pop!_OS 24.04 on Asus C302CA Chromebook (aka Cave)
---

I love the hardware, but Google stopped supplying updates for this model.
So I installed Pop!_OS 24.04 on it, and it works great.
Even the speakers work, which was not true when I tested the previous version of the OS.

It has 4GB RAM and the installer configured an 8.3GB swap file.
This has been fine.

I enabled full-disk encryption, so this is a great travel laptop now.

These are the high-level steps I followed:
1. Remove firmware write-protect screw under black pad in the center bottom of mainboard.
2. Put device in Developer Mode.
3. Flash [Mr. Chromebox UEFI firmware](https://mrchromebox.tech/#fwscript).
4. Reboot.
5. [Install the OS](https://docs.chrultrabook.com/docs/installing/installing-linux.html). Ignore the part about fixing the audio.
