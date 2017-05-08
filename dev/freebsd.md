---
layout: page
title: FreeBSD
---


* see *Absolute FreeBSD* for good overview

# Install

Install public key of ansible host to target freebsd machine:

        ssh-copy-id smeredith@freebsd-host

Use `visudo` to allow members of `wheel` to sudo w/o password:

        $ su
        # pkg install sudo
        # visudo

Uncomment:

    %wheel ALL=(ALL) NOPASSWD: ALL

1.  sudo pkg install python
2.  security patches (see below)
3.  upgrade packages (see below)
4.  use ansible to finish the configuration
5.  take a snapshot

* To map caps-lock to ctrl in X, use the UI keyboard settings.
* Connect to wifi. See \<www.bitmine.org\>.
* Enble screen locker via GUI.

# Update

## pkg

    # pkg upgrade

## system

### security patches

    # freebsd-update fetch
    # freebsd-update install

### minor versions

See <https://www.freebsd.org/doc/handbook/updating-upgrading-freebsdupdate.html>

Installed packages will continue to work.

### major versions

See <https://www.freebsd.org/doc/handbook/updating-upgrading-freebsdupdate.html>

Installed packages will require forced upgrade.

# defaults

* system keeps configuration in `defaults` sub dirs
* you override default config in seperate file of same name
* system updates the config in `defaults` so you don't have to merge
* don't copy defaults and edit; empty config file means no changes to default

# sudo

Use `visudo` to allow members of `wheel` to sudo su pkg install sudo visudo:

      userX ALL=(ALL) ALL

# Setup

## Raspberry Pi

* first boot takes a little while due to auto resize of the disk
* once resize is done, you can ssh as freebsd, the su to root
* default user: freebsd/freebsd
* default root password: root
* use `adduser` to add user smeredith and add to group `wheel` to allow su to root
* `rmuser freebsd`
* change root password using `passwd`
* change hostname in `/etc/rc.conf`

## General

Use `visudo` to allow members of wheel to sudo w/o password:

      $ su
      # pkg install sudo
      # visudo

Install python to allow ansible to run:

      # sudo pkg install python

Install public key of ansible host to target FreeBSD machine:

      $ ssh-copy-id -i ~/.ssh/id_rsa.pub freebsd-host

Do the rest via ansible.

### Add NTFS USB Drive

Install NTFS driver:

      # pkg install fusefs-ntfs
      # sysrc kld_list+=fuse

Add to `/etc/rc.conf`:

      fuse_enable="yes"

`ntfs/ssd` is an automatic label for this disk.

To mount for 1-time use:

      # ntfs-3g /dev/ntfs/ssd /mnt/ssd

I could not get these to mount at boot; hang. I tried adding the following to `/etc/fstab`:

      UUID=B25ECD4B5ECD08D5 /mnt/ssd ntfs-3g  rw,sync,suid,dev,exec,auto,nouser,async,dmask=000,fmask=111,utf8=1,late      0       2
      UUID=9C0AD9830AD95B3A /mnt/bighdd ntfs-3g rw,sync,suid,dev,exec,auto,nouser,async,dmask=000,fmask=111,utf8=1,late      0       2

### Mount a SMB Drive

Need to be su

      # mount_smbfs -N -u <uid> -g <gid> -I 192.168.0.2 //guest@192.168.0.2/shared /smb/shared
      # mount_smbfs -N -u <uid> -g <gid> -I 192.168.0.2 //guest@192.168.0.2/shared /smb/shared

* `-N` is to prevent prompting for password
* get \<uid\> and \<gid\> from `id` command

# Misc

* network speed test
* reciever: `nc -l 3333 > /dev/null`
* sender: `dd if=/dev/zero bs=64k count=2000 | nc -v 192.168.0.18 3333`
* (7112319 bytes/sec) from mac to Raspberry Pi 2 running FreeBSD via ethernet
* (11435232 bytes/sec) from mac to Raspberry Pi 2 running Raspian via ethernet
* restart a service: `service <service-name> restart`

# Directory structure

* `/usr/local/etc`: app config files
* `/usr/local/share/doc`: app docs
* `/usr/local/etc/rc.d`: app startup scripts

# Files

* `/etc/rc.conf`: startup options
* `/var/log/messages` system log
* `/var/run/dmesg.boot` boot messages
* `/boot/loader.conf` loader settings, boot-time tunables, including kernel sysctl values

# Recovery

* run `passwd` in single user mode to reset password for root
* edit fstab in single user mode to resolve boot disk issues
* edit `/etc/rc.conf` to disable startup app that panics and prevents system from booting

# Dell Inspiron E1505

Broadcom BCM4311 802\.11b/g WLAN (driver: bwn).
The wifi keyboard shortcut works (if enabled in BIOS), but does not change the wifi light on the keyboard one time, need to install new wifi firmware: (download the sources tar first)

    fetch ftp://ftp.freebsd.org/pub/FreeBSD/releases/386/11.0-RELEASE/src.txz
    tar -C / -xzvf src.txz
    cd /usr/ports/net/bwn-firmware-kmod
    make install clean

add this to /boot/loader.conf

    if_bwn_load="YES"
    bwn_v4_ucode_load="YES"

add the following to /etc/rc.conf

    wlans_bwn0="wlan0"
    ifconfig_wlan0="WPA DHCP"

* pressing the "Wifi Up/Down" button in wifimgr adds/removes wlan0, probably using rc.conf
* keyboard wifi light goes off when the interface is removed
* can't make this work: disconnecting ethernet then connecting to wifi
* boot with ethernet disconnected
* add to `/etc/wpa_supplicant.conf` for AP configs.


      network={
          ssid="myssid"
          psk="mypsk"
      }

Normal operation:

* use ethernet at desk: disable wifi via keyboard shortcut
* use wifi: enable via shortcut, disconnect wifi, and reboot
* use wifimgr to connect to new AP (writes to wpa\_supplicant.conf and restarts network)
* `service netif restart` when switching between wireless and wifi

# Desktop Environments

## Display Manager

* see <https://cooltrainer.org/a-freebsd-desktop-howto/>
* graphical login program
* KDM, GDM, slim
* if you use KDM or GDM, they will launch the corresponding session, and so exec is required in .xinitrc
* if you use slim, it will startx so you need to exec the session in .xinitrc
* a DM can be used to switch between desktop environments or window managers

## file: .xinitrc

* executed when xstart is called
* runs until x session exits
* can run some apps in the backgroun by launching them before `exec foo-wm`

# Jekyll

    pkg install ruby22-gems
    gem install jekyll
    gem install github-pages
    # local build / test for errors
    jekyll build --incremental --drafts
    # local preview
    jekyll serve --host 0.0.0.0 --port 4000 --drafts --watchs

# Burn and img to SD card

Assuming sd card in USB adapter:

    gzip -dc ~/Downloads/LibreELEC-RPi2.arm-7.0.2.img.gz | sudo dd of=/dev/da0 obs=64k

# zfs

## datasets

    zfs get all <pool>

* a stub is a parent of other datasets

## snapshot

Create snapshot of all datasets:

    zfs snapshot -r zroot@hello-snapshot

Roll back to snapshot (no -r; must do for each dataset):

    zfs rollback zroot@hello-snapshot

Merge snapshot back in and remove snapshot:

    zfs destroy -r zroot@hello-snapshot

List snapshotes:

    zfs list -t snapshot

# 7-zip

Install and use peazip.

# Printers

Set up using KDE Add Printer.

Brother HL-3170CDW \* \<ipp://192\.168\.0\.15/ipp/port1\> \* Generic PCL 6 driver \* Only B\&W

Brother 1440 \* \<ipp://192\.168\.0\.18:631/printers/Brother\_HL-1440\_series\>

# Scanners

See <https://forums.freebsd.org/threads/54100/> and <https://www.freebsd.org/doc/handbook/scanners.html>

In `/etc/rc.conf`:

    devfs_system_ruleset="system"


In `/etc/devfs.rules`:

    [system=5]
    add path ugen1.2 mode 0660 group usb
    add path usb/1.2.0 mode 0666 group usb

# Compaq nc8430

## Suspend and Resume

* `acpiconf -s 3` to suspend
* S4 doesn?t work, with or without `hw.acpi.s4bios`
* `hw.acpi.lid_switch_state=S3` to enable lid switch to suspend
* `hw.acpi.reset_video` must be 0 for suspend to work

