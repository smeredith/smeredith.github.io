---
layout: post
title: Installing FreeBSD 11 on a Dell Inspiron E1505
---

I recently installed FreeBSD 11.0-RELEASE on an old Dell Inspiron E1505.
I made these notes on the process in order to help you should you want to do the same.

## Installation

This machine has a 32-bit processor and 2GB of RAM.
It requires the i386 installer image.
I used FreeBSD-11.0-RELEASE-i386-disc1.iso to create a CD and installed from that.

I needed a wired network connection to complete the installation, so I attached to an ethernet network before performing the installation.

In the "Distribution Select" dialog, I added "src" to the already selected "ports" option.
The primary reason for this was so that I could later build the WiFi card firmware, which requires these file.

In the "Partitioning" dialog, I selected "Auto (ZFS)" and took the defaults.
"Auto (UFS)" will also work.

I selected "sshd," "ntpd", "powerd", and "dumpdev" at the "System Configuration" dialog.

I ended up with a 2 GB swap file, which is fine.

## Desktop Environment

I installed and tested MATE, Xfce, KDE 4, and Cinnamon.
Cinnamon is too slow to use, but the others are fine.

## WiFi

**NOTE:** _The keyboard hotkey to toggle WiFi (function-F2) works in FreeBSD, but it does not toggle the keyboard WiFi LED the way it does on Windows.
If you can't get WiFi to work, try this hotkey to see if it might be disabled.
You can disable the hotkey via the BIOS if you wish._

The WiFi card in this laptop is a Broadcom BCM4311 802.11b/g WLAN.
The correct driver for this is "bwn".

### Update the firmware

First, I updated the firmware on the WiFi card.
This didn't seem to affect the operation of the card under Windows when I later booted to Windows.

To update the firmware, kernel sources are required.
If you didn't check the "src" box during installation, you can download the version that matched your installed version and extract the files:

    $ fetch ftp://ftp.freebsd.org/pub/FreeBSD/releases/386/11.0-RELEASE/src.txz
    $ tar -C / -xzvf src.txz

You don't need to build the kernel, but the files are required for the next step, building and installing the WiFi firmware:

    # cd /usr/ports/net/bwn-firmware-kmod
    # make install clean

### Enable WiFi

I added these two lines to `/boot/loader.conf`:

    if_bwn_load="YES"
    bwn_v4_ucode_load="YES"

I added the following two lines to `/etc/rc.conf`

    wlans_bwn0="wlan0"
    ifconfig_wlan0="WPA DHCP"

This is because I am using WPA and DHCP.

I created a file named `/etc/wpa_supplicant.conf` and added my SSID and password:

    network={
        ssid="my_SSID"
        psk="my_password"
    }

I unplugged the ethernet cable and rebooted.
When you reboot with these settings, you should see the WiFi LED on the keyboard light up.
The LED is lit when the interface is "up," even if WiFi is disabled by the keyboard shortcut.

You should see an IPv4 address when you use:

    $ ifconfig wlan0

### Connect to a different AP

I installed the GUI app "wifimgr" to connect to other WiFi access points:

    # pkg install wifimgr

## Browser

I installed Chromium and Firefox.

Chromium has a bug which make it unusable: about half the time, loading a page hangs a tab.
The only way to recover is to kill the tab.

Firefox works correctly.
It is also faster and uses less memory than Chromium.

## Performance

Performance is surprisingly good.
Booting talkes longer than it does on Windows.
YouTube in HD is a weak point: the machine has difficulty keeping up.
But is no different than how the machine behaves with Windows installed.
I think the CPU just isn't fast enough.

The HDD is also limiting factor.
I replaced it with an SSD and performance is even better.
The laptop is also quieter because the fan near the disk does not turn on.
