---
layout: post
title: Installing FreeBSD 11 on a Dell Inspiron E1505
---

I recently installed FreeBSD 11.0-RELEASE on an old Dell Inspiron E1505.
I made these notes on the process in order to help you should you want to do the same.

## Installation

This machine has a 32-bit Core Duo processor and 2GB of RAM.
It requires the i386 installer image.
I used FreeBSD-11.0-RELEASE-i386-disc1.iso to create a CD and installed from that.

I needed a wired network connection to complete the installation, so I attached to an ethernet network before performing the installation.

In the "Distribution Select" dialog, I added "src" to the already selected "ports" option.
As discussed below, I needed to do this in order to get WiFi working

In the "Partitioning" dialog, I selected "Auto (ZFS)" and took the defaults.
"Auto (UFS)" also works.

I selected "sshd," "ntpd", "powerd", and "dumpdev" at the "System Configuration" dialog.

I ended up with a 2 GB swap file by default, which is fine.

## Desktop Environment

I installed and tested MATE, Xfce, KDE 4, and Cinnamon.
Cinnamon is too slow to use, but the others are fine.

## WiFi

**NOTE:** _The keyboard hotkey to toggle WiFi (fn-F2) works in FreeBSD, but it does not toggle the keyboard WiFi LED the way it does on Windows.
It is impossible to know what state it's in.
If you can't get WiFi to work, try this hotkey to see if WiFi might be disabled.
Or, you can disable the hotkey altogether via the BIOS to remove this ambiguity._

The WiFi card in this laptop is a Broadcom BCM4311 802.11b/g WLAN.
The correct driver for this is "bwn".

### Update the firmware

First, I updated the firmware on the WiFi card.
This didn't seem to affect the operation of the card under Windows when I later booted to that OS.

To update the firmware, kernel sources are required.
If you didn't check the "src" box during installation, you can download the version that matched your installed version and extract the files:

    # fetch ftp://ftp.freebsd.org/pub/FreeBSD/releases/386/11.0-RELEASE/src.txz
    # tar -C / -xzvf src.txz

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

The last line is because my AP is configured for WPA and DHCP.

I created a file named `/etc/wpa_supplicant.conf` and added my SSID and WiFi password:

    network={
        ssid="my_SSID"
        psk="my_password"
    }

I unplugged the Ethernet cable and rebooted.
When you reboot with these settings, you should see the WiFi LED on the keyboard light up.
The LED is lit when the interface is "up," even if WiFi is disabled by the keyboard shortcut.

I knew this was working when I saw an IPv4 address assigned to WLAN0 when I checked:

    $ ifconfig wlan0

### Connect to a different AP

I installed the GUI app "wifimgr" and use it to connect to other WiFi access points:

    # pkg install wifimgr

## Browser

I installed Chromium and Firefox.

Chromium has a bug which make it unusable: about half the time, loading a page hangs a tab.
The only way to recover is to kill the tab.

Firefox works correctly.
It is also faster and uses less memory than Chromium.

## Performance

Performance is surprisingly good for such an old machine.
Booting takes longer than it does on Windows.
YouTube in HD is a weak point: the machine has difficulty keeping up.
But is no different than how the machine behaves under Windows.
I think the CPU just isn't fast enough.

The HDD is also limiting factor.
I replaced it with an SSD and performance is even better.
The laptop is also quieter because the fan near the disk does not turn on.

## What else works

* The trackpad works fine, though there are no multi-finger gestures.
* The hotkeys for volume control work.
* Some of the fn number pad keys (/, *, -, +) work.
* The fn brightness controls on the up and down arrow keys work.
* The DVD drive works.
* The speakers.
* The headphones jack.

## What doesn't work

* Suspend/resume does not work.
* Hibernate does not work.
* A microphone plugged in to the mic jack does not work.
* The mute keyboard shortcut does not work.
* Some of the fn number pad keys (7, 8, 9, 4, 5, 6, 1, 2, 3, 0, .) don't work.

It is possible that I just don't know how to configure these things.

## What I did not try

* The PC Card slot.
* The SD card slot.
* USB.
* Firewire.
* The modem.
* The VGA port.

## Conclusion

I've only had FreeBSD installed for a couple of weeks, but this is a completely usable laptop.
With the SSD, it is usually completely silent.

If you have this setup yourself, I'd love to hear your experience.