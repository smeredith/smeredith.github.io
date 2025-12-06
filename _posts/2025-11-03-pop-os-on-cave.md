---
layout: post
title: Installing Pop!_OS 24.04 on Asus C302CA Chromebook (aka Cave)
---

I love the hardware, but Google stopped supplying updates for this model.
So I installed Pop!_OS 24.04 on it, and it works great.

The touchscreen is very bright and readable.
The camera works fine.
The sound is pretty good, but the microphone does not work.
The battery life is good enough, considering its age.
Bluetooth works.
The touchpad works.

It has 4 GB RAM and the installer automatically configured an 8.3 GB swap file.
This has been fine.
Don't expect miracles, but browsing with Chrome works at least as well as it did on the machine when it had ChromeOS.
And I enjoy flipping the keyboard around and reading PDFs in portrait mode like a magazine.

I enabled full-disk encryption, so this is a great travel laptop now.
I can even charge it with a low-wattage 5V USB-A phone charger or portable battery pack if I forget to pack a more powerful charger.

Beside the microphone not working, the only other problem I have encountered is that once in a while, the device does not come out of suspend properly when I open the lid.
I see a gray screen and a working mouse cursor, but I do not see the login screen as expected.
To recover, restart the COSMIC greeter from the console:
- Switch to a TTY (Ctrl+Alt+F2).
- `sudo systemctl restart cosmic-greeter`.
- Switch back to the GUI session (Ctrl+Alt+F1).

These are the high-level steps I followed to install the OS:
1. Remove firmware write-protect screw under black pad in the center bottom of mainboard.
2. Put device in Developer Mode.
3. Flash [Mr. Chromebox UEFI firmware](https://mrchromebox.tech/#fwscript).
4. Reboot.
5. [Install the OS](https://docs.chrultrabook.com/docs/installing/installing-linux.html). Ignore the part about fixing the audio.
