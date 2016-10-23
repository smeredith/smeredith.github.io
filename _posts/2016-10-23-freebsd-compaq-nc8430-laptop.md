---
layout: post
title: Installing FreeBSD 11 on a Compaq nc8430
---

I recently installed FreeBSD 11.0-RELEASE on an old Compaq nc8430 laptop.
I made these notes on the process in order to help you should you want to do the same.

## Installation

This machine has a 64-bit processor and 2GB of RAM.
While you can install the i386 version, the amd64 installer image is a better choice.
I used FreeBSD-11.0-RELEASE-amd64-disc1.iso to create a CD and installed from that.

I needed a wired network connection to complete the installation, so I attached to an ethernet network before performing the installation.

In the "Distribution Select" dialog, I added "src" to the already selected "ports" and "lib32" options.

In the "Partitioning" dialog, I selected "Auto (ZFS)" and took the defaults.
"Auto (UFS)" will also work.

I selected "sshd," "ntpd", "powerd", and "dumpdev" at the "System Configuration" dialog.

I ended up with a 2 GB swap file by default, which is fine.

## Desktop Environment

I installed and tested MATE, Xfce, and KDE 4.
They all work well.

## WiFi

I added the following two lines to `/etc/rc.conf`

    wlans_0="wlan0"
    ifconfig_wlan0="WPA DHCP"

This is because my AP is configured for WPA and DHCP.

I created a file named `/etc/wpa_supplicant.conf` and added my SSID and WiFi password:

    network={
        ssid="my_SSID"
        psk="my_password"
    }

I unplugged the Ethernet cable and rebooted.
When you reboot with these settings, you should see the blue WiFi LED on the keyboard light up.
The keyboard button may be used to toggle WiFi, and the LED will refect the current state.

You should see an IPv4 address when you use:

    $ ifconfig wlan0

and status should be "associated".

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

Performance is good.
Booting takes longer than it does on Windows.
YouTube struggles with 1080P, but handles 720P in full screen mode.

## What else works

* The mute button works, though the actual operation sometimes lags the press by a few seconds.
* The track pad works.
The right edge can be used for scrolling.
* The blue mouse pointer stick works to move the mouse.
* The speakers work.

## What doesn't work

* Suspend to RAM does not work (no keyboard or trackpad when resuming.)
* Hibernate does not work (the screen goes black but the machine never powers down.)
* The volume buttons do not work.
* None of the blue fn keys, like brightness control, work.
* The fan seems to stay on maximum all the time, making a lot of noise.

It is possible that I just don't know how to configure these things.

## Things I didn't try

* USB.
* Firewire.
* The PC Card slot.
* The SD card slot.
* The VGA port.
* The serial port.
* The modem.
* The mic jack.
* The headphone jack.

## Conclusion

Functionally, the laptop is usable.
However, the constant noise from the fan is very loud and makes this an unpleasant laptop to use for long.
If you can stand that, you might be in good shape.

I'd love to hear from you if you are using this setup.
