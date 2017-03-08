---
layout: post
title: Installing FreeBSD 11 on a Compaq nc8430
---

I recently installed FreeBSD 11.0-RELEASE on an old Compaq nc8430 laptop from 2007.
I made these notes on the process in order to help you should you want to do the same.

## Installation

This machine has a 64-bit processor and 2GB of RAM.
I installed the amd64 version.
I used FreeBSD-11.0-RELEASE-amd64-disc1.iso to create a CD and installed from that.

I needed a wired network connection to complete the installation, so I attached to an ethernet network before performing the installation.

In the "Distribution Select" dialog, I added "src" to the already selected "ports" and "lib32" options.

In the "Partitioning" dialog, I selected "Auto (ZFS)" and took the defaults.
"Auto (UFS)" will also work.
I encrypted the drive using GELI, including the swap partition.

I selected "sshd," "ntpd", "powerd", and "dumpdev" at the "System Configuration" dialog.

I entered 4 GB of swap space.

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

### Switching Between WiFi and Ethernet

I don't know of a good way to switch between network interfaces when I connect or disconnect the network cable, so I just reset the networking service and let it sort itself out:

    # service netif restart

## Browsers

I installed Chromium and Firefox.

Chromium has a bug which make it unusable: about half the time, loading a page hangs a tab.
The only way to recover is to kill the tab.

Firefox works correctly.
It also uses less memory than Chromium.

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
* USB works.
* The DVD drive works for reading and creating disks.
* The external VGA connector works.
* Suspend to RAM and resume works.
I needed to add `/etc/rc.d/moused` to `/etc/rc.resume` in order for the mouse to work upon resume.

## What doesn't work

* Hibernate does not work (the screen goes black but the machine never powers down.)
* The volume buttons work intermittently.
* None of the blue fn keys, like brightness control, work.
* I see a lot of messages like this to the console:
```
    acpi_tx0: _CRT value is absurd, ignored (256.1C)
```
## Things I didn't try

* Firewire.
* The PC Card slot.
* The SD card slot.
* The serial port.
* The modem.
* The mic jack.
* The headphone jack.

## Conclusion

Functionally, the laptop is usable, and it's pretty fast, even with KDE.
However, the constant noise from the fan is very loud and makes this an unpleasant laptop to use for long.
If you can stand that, you might be in good shape.

I'd love to hear from you if you are using this setup.

## Update

I've upgraded the machine from 2GB to 4GB of RAM.
The machine only sees 3.3GB.
I don't think that is a function of the OS because this is what the BIOS reports upon boot.
