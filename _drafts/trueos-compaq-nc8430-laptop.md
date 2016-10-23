---
layout: post
title: Installing TrueOS 11 on a Compaq nc8430
---

I recently installed TrueOS 11.0 an old Compaq nc8430 laptop.
I made these notes on the process in order to help you should you want to do the same.

## Installation

This machine has a 64-bit processor and 2GB of RAM.

I first used TrueOS-Desktop-2016-10-14-x64-USB.img to create a USB boot drive and tried to install from that.
The laptop would not finish booting to the installer.

Then I downloaded TrueOS-Desktop-2016-10-14-x64-USB.iso and burned a DVD and used that to install.
Installing using the "Start graphical install (Auto detect video)" didn't work due to masssive flicker.
The "vesa" option worked.



