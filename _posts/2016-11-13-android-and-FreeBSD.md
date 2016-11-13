---
layout: post
title: Connecting Android to FreeBSD 11
---

I wanted to connect my Nexus 5 to my laptop running FreeBSD 11.
My goal was to be able to move files from the phone to the laptop.
I did not find clear instructions on how to make that work, so I will share what I did.

## simple-mtpfs

First, each time I connect my phone to a computer, it gives me a notification that it is only enabled for changing.
In order to enable file transfers, I click the notification and selecte MTP.

I tried a fuse driver named `mtpfs` but I was not able to make it work.
So then I installed `simple-mtpfs` and got that working.

As root:

    # pkg install fusefs-simple-mtpfs

Then I created a mount point for the Android file system:

    # mkdir /mnt/android
    # chown user:user /mnt/android

where "user" is the name of the non-root user account I use.

Then I added a line to `/boot/loader.conf` to allow the OS to use fuse file systems:

    fuse_load="YES"

At this point I restarted the system.

In order to browse the connected Android phone filesystem as "user", I need to mount it as root.
This is inconvenient: I want to be able to mount it as "user".
But I can't figure out how to do that.

So, as root:

    # simple-mtpfs /mnt/android -o allow-other

Now, as "user" I can use `/mnt/android` as any other file system.

## KDE Connect

I also installed and played with KDE Connect.
This is interesting, and most features seemed to work, but I could not get its remote file system browsing over Wi-Fi to function.
I installed `sshfs` because it complained that it was missing, but that did not help.
Since the thing I want most from this is the remote file browsing, I probably won't ever use KDE Connect.
