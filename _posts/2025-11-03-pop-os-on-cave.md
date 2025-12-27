---
layout: post
title: Installing Pop!_OS 24.04 on Asus C302CA Chromebook (aka Cave)
---

I love the hardware, but Google has stopped supplying updates for this model.
As a result, I installed Pop!_OS 24.04 on it, and it works great.

The touchscreen is very bright and readable.
The camera works fine.
The sound quality is good, although the microphone does not work.
Battery life is acceptable, considering the device’s age.
Bluetooth works.
The touchpad works.
It has no fan, so it is very quiet.

The device has 4&nbsp;GB of RAM, and the installer automatically configured an 8.3&nbsp;GB swap file.
This setup has been sufficient.
Do not expect miracles, but browsing with Chrome works at least as well as it did when the machine was running ChromeOS.
I also enjoy flipping the keyboard around and reading PDFs in portrait mode, like a magazine.

I enabled full-disk encryption, which makes this a great travel laptop.
I can even charge it with a low-wattage 5&nbsp;V USB-A phone charger or a portable battery pack if I forget to bring a more powerful charger.

Aside from the nonfunctional microphone, the only other issue I have encountered is that, occasionally, the device does not resume properly from suspend when I open the lid.
I see a gray screen with a working mouse cursor, but the login screen does not appear.
However, if I enter my password blindly, the system logs in and the normal desktop appears.

These are the high-level steps I followed to install the operating system:

1. Remove the firmware write-protect screw under the black pad in the center-bottom of the mainboard.
2. Put the device into Developer Mode.
3. Flash the [MrChromebox UEFI firmware](https://mrchromebox.tech/#fwscript).
4. Reboot.
5. [Install the OS](https://docs.chrultrabook.com/docs/installing/installing-linux.html). Ignore the section about fixing the audio.
