---
layout: post
title: Installing FreeBSD on a Dell Inspiron E1505
---

I recently install FreeBSD 11.0-RELEASE on an old Dell Inspiron E1505.
I made these notes on the process in order to help you should you want to do the same.

## Installation

This machine has a 32-bit processor and 2GB of RAM.
It requires the i386 installer image.
I used the disc1.iso to create a CD and installed from there.

You'll need a wired network connection to complete the installation, so attach an ethernet connection before performing the installation.

In the "Distribution Select" dialog, I added "src" to the already selected "ports" option.
The primary reason for this was so that I could build the WiFi card firmware, which requires these file.

In the "Partitioning" dialog, I tried both "Auto (UFS)" and "Auto (ZFS)" and took the defaults for each.
Either will work.

## Desktop Environment

I installed and tested MATE, Xfce, KDE 4, and Cinnamon.
Cinnamon is too slow to use, but the others are fine.

## WiFi

Be aware that the keyboard hotkey to toggle WiFi (function-F2) works in FreeBSD, but it does not toggle the keyboard WiFi LED the way it does on Windows.
If you can't get WiFi to work, try this hot key to see if it might be disabled.

### Update the firmware

First, you need to update the firmware on the WiFi card.
This didn't seem to affect the operation of the card under Windows so if you need to reinstall that OS in the future, it should be fine.

To update the firmware, you need the kernel source.
If you didn't check the "src" box during installation, you can download the version that matched your installed version and extract the files:

    fetch ftp://ftp.freebsd.org/pub/FreeBSD/releases/386/11.0-RELEASE/src.txz
    tar -C / -xzvf src.txz

You don't need to build the kernel, but the files are required for the next step, building and installing the WiFi firmware:

    cd /usr/ports/net/bwn-firmware-kmod
    make install clean

### Enable WiFi

Add these two lines to `/boot/loader.conf`:

    if_bwn_load="YES"
    bwn_v4_ucode_load="YES"

### Connect
## Browser

I installed Chromium and Firefox.

Chromium has a bug which make it unusable: about half the time, loading a page hangs a tab.
The only way to recover is to kill the tab.

Firefox appears to work correctly.
It is also faster and uses less memory and Chromium.

## Performance

Performance is surprisingly good.
YouTube in HD is a weak point: the machine has difficulty keeping up.
But is no different than how the machine behaves with Windows installed.
I think the CPU just isn't fast enough.

The HDD is also limiting factor.
I replaced it with an SSD and performance is even better.
