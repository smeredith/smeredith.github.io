---
layout: page
title: Linux
---

* Path is set in `~/.profile`. Export not required.
* Dictionary file: `/usr/share/dict/words`. To install: `sudu apt-get install wamerican`.
* Allowable chars for filenames: \[a-zA-Z0-9-\_.\].
* To work with Windows on a network, limit hostname to 15 characters or less.

## Hyper-V Disk Scheduler

Noop is the best disk scheduler for VHDs. In `/etc/default/grub` change

    GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"

to

    GRUB_CMDLINE_LINUX_DEFAULT="quiet splash elevator=noop"

The contents of the "" may vary. Then run:

    $ sudo update-grub2

Restart. Check the scheduler like this:

    $ cat /sys/block/sda/queue/scheduler
    [noop] deadline cfq

The selected scheduler will be in square brackets.

## SSH

### Configuration

* System-wide client configuration is in `/etc/ssh/ssh_config`.
* System-wide daemon configuration is in `/etc/ssh/sshd_config`.

Per-user configuration is in `~/.ssh/config`. Can specify multiple hosts. Example contents:

    Host alias
    HostName my-server.com
    User admin
    IdentityFile ~/.ssh/admin.id_rsa

Disable password login, edit `/etc/ssh/sshd_config`:

    PasswordAuthentication no

Create an ssh key on the client:

    $ ssh-keygen

This will place the private key in `~/.ssh/id_rsa` and the public key in `~/.ssh/id_rsa.pub`.

Copy that public key to a remote server (need to `brew install ssh-copy-id` on mac first):

    $ ssh-copy-id user@example.com

To do it manually, append the contents of `~/.ssh/id_rsa.pub` to the end of `~/.ssh/authorized_keys` on the remote machine.

    $ cat ~/.ssh/id_rsa.pub | ssh host 'cat >> ~/.ssh/authorized_keys'

### Usage

To execute a single command on host:

    $ ssh host command

## Clipboard

You can use the x clipboard over ssh if X forwarding is in use (use -Y) and one x app is running.

To forward X:

    $ ssh -Y host

`-X` also works, but `-Y` is less restrictive and allows clipboard sync.
If you copy something into the X clipboard on the remote, the local
clipboard can sync (if the X server is configured correctly I guess.)
Use `xclip` on the remote machine to get something on the clipboard. If
using Vim with `+xterm_clipboard` on the remote, then `*` will copy to
the x-clipboard. From SSH, I could only get this to work if at least one
X app is running.

## Samba

To share, edit /etc/samba/smb.conf.

Mount a windows dir:

    $ sudo apt-get install cifs-utils
    $ sudo mount -t cifs //raspberrypi/photos /mnt/photos
    $ sudo mount -t cifs //stmer6/MapsRepoBackup /mnt/mapsbackup -o username=stmer,domain=REDMOND,iocharset=utf8,file_mode=0777,dir_mode=0777

or edit `/etc/fstab` and add this line:

    //stmer6/MapsRepoBackup /mnt/mapsbackup cifs credentials=/home/smeredith/.smbcredentials,iocharset=utf8,sec=ntlm,uid=1000,gid=1000,dir_mode=0777,file_mode=0777 0 0

where `uid` and `gid` are found using the `id` command. These are the owner and group for the share.

where `.smbcredentials` is:

    username=stmer
    password=<domain_password>
    domain=REDMOND

To restart:

    $ sudo service restart smbd
    $ sudo service restart nmbd

or on Raspian:

    $ sudo service samba restart

## davfs

    $ sudo apt-get install davfs2
    $ sudo mkdir /mnt/sme
    $ sudo mount -o uid=pi -o gid=pi -t davfs https://webdav.storagemadeeasy.com /mnt/sme

Add a line to `/etc/davfs2/secrets`:

    /mnt/sme stemeredith <password>

Add yourself, and all users who need to be able to mount a cloud drive, to the group `davfs2`:

    $ sudo usermod -aG davfs2 pi

Now you?ll have to give normal users the permission to run `davfs2`, without the need for root privileges:

    $ sudo dpkg-reconfigure davfs2

Add the next line to `/etc/fstab`:

    https://webdav.storagemadeeasy.com /mnt/sme  davfs rw,noexec,noauto,user,async,uid=pi,gid=pi  0  0

To mount:

    $ mount /mnt/sme

Or, to make it mount on boot automatically, change `/etc/fstab`:

    https://webdav.storagemadeeasy.com /mnt/sme  davfs rw,noexec,_netdev,auto,user,async,uid=pi,gid=pi  0  0

The `_netdev` makes boot wait until network is available.

### rsync

Sync to make two directories the same:

    $ rsync -avu --delete /media/Big/photos/ /mnt/sme/onedrive/Pictures/Mirror
    $ rsync -avu --delete "/media/Big/photos/2015/2015 Jan" /mnt/sme/onedrive/Pictures/Mirror/2015

* `-r` recursive
* `-u` update only newer files
* `-v` verbose
* `--delete` delete files on the destination not on the source.
* `-n` dry run

Move files from one directory to another:

    $ rsync -v --remove-source-files /mnt/sme/onedrive-jessi/Pictures/Camera\ Roll/* /mnt/bighdd/documents/Jessica/photos/camera-roll

## Bash

* Go to beginning of line: ctrl-a
* Go to end of line: ctrl-e
* Delete line from end: ctrl-u
* Clear line after cursor: ctrl-k
* Go forward one word: alt-f
* Go back one word: alt-b
* Cancel current line: ctrl-c
* Clear screen: ctrl-l

Or `set -o vi` to turn on vim key bindings in Bash\!

To run a command in a loop:

    RET=0; while [ $RET -eq 0 ]; do legacytest; RET=$?; done

### bashrc

Make customizations here. Includ this file from .bash\_profile.

## Tar/Zip

Make `tar.gz`

    $ tar -czf output.tar.gz input

Extract `tar.gz`

    $ tar -xzf filename.tar.gz

Extract `tar.bz2`

    $ tar -jxf filename.tar.bz2

## OpenVPN

Host: `stmerubus14.cloudapp.net`\
SSH password auth has been disabled on this machine. Both `ubrick` and
`blacktop` have private keys that can be used to login. Server config in
`/etc/openvpn`.

To use OpenVPN over T-Mobile, either use the TCP profile or disable IPv6
in the Access Point settings. I do not have UDP6 specified in any config
OpenVPN config file on the client or server, but it is choosing that
protocol. Tried this script on the server:

    #!/bin/bash
    echo net.ipv6.conf.all.disable_ipv6 = 1 >> /etc/sysctl.conf
    echo net.ipv6.conf.default.disable_ipv6 = 1 >> /etc/sysctl.conf
    echo net.ipv6.conf.lo.disable_ipv6 = 1 >> /etc/sysctl.conf
    sysctl -p

Disabling IPV6 in Linux didn't make a difference: OpenVPN still connected
using UDP6\. Changed the APN settings on the phone worked.

## AWS VPN Server

See <https://github.com/sarfata/voodooprivacy> and
<https://www.webdigi.co.uk/blog/2015/how-to-setup-your-own-private-secure-free-vpn-on-the-amazon-aws-cloud-in-10-minutes/>.
The later requires a default VPC, which for old accounts only exists for
regions you have never used before. For me, non of the US regions would
work. San Paulo works for me. Ireland, Frankfurt, and Sydney all have
default VPCs and should work. See
<https://aws.amazon.com/vpc/faqs/#Default_VPCs> for more info about
default VPCs. Another option is to create a new AWS account just for
this. You need to set it up each time you want to use it and delete the
stack when finished. I couldn?t get the VPN to connect using T-Mobile
LTE. I was successful with PPTP over Wi-Fi.

## Add a Swap File

You can have more than one swap file plus a swap partition. Use `free` to check the swap status.

Create and add a swap file:

    $ sudo dd if=/dev/zero of=/10gb.swap bs=1G count=10

    $ sudo chmod 600 /10gb.swap
    $ sudo mkswap /10gb.swap
    $ sudo swapon /10gb.swap

Edit `/etc/fstab` to add:

    /10gb.swap  none  swap  sw  0 0

## Add New VHD

Find the new vhd:

    $ sudo lshw -C disk
    *-disk:3
        description: SCSI Disk
        physical id: 0.0.4
        bus info: scsi@3:0.0.4
        logical name: /dev/sde
        size: 50GiB (53GB)
        configuration: sectorsize=4096

Partition the disk:

    $ sudo fdisk /dev/sde

1.  select "n" (for new partition)
2.  select "p" (for primary)
3.  select "1" (for partition number 1\)
4.  select "w" (to write the partition)

Format the partition:

    $ sudo mkfs -t ext3 /dev/sde1

Create mount point:

    $ sudo mkdir /mnt/build3

Find UUID of vhd (you can also find it in the output of mkfs):

    $ ls -laF /dev/disk/by-uuid/
    lrwxrwxrwx 1 root root  10 Mar 25 08:43 7113c06c-a549-4793-b4fc-686f8d23b537 -> ../../sde1

Update `/etc/fstab` with UUID of new vhd:

    UUID=7113c06c-a549-4793-b4fc-686f8d23b537   /mnt/build3 ext3    defaults    0   2

or for a Windows drive:

    UUID=9C0AD9830AD95B3A /media/Big ntfs-3g    rw,sync,suid,dev,exec,auto,nouser,async,dmask=000,fmask=111,utf8=1    0       2

Mount it:

    $ sudo mount -a

Own it:

    $ sudo chown -R smeredith:smeredith /mnt/build3

## Photos

To add keywords to the metadata for a file:

    $ exiftool -keywords+=<new-keyword> file.jpeg
    $ exiftool -keywords+=<new-keyword> -keywords+=<another-keyword> file.jpeg

To dump the keywords for a file:

    $ exiftool -keywords file.jpeg

To set the date of a scanned photo:

    $ exiftool "-AllDates=1988:01:01 00:00:00" -overwrite_original file.jpeg

To add a description:

    $ exiftool -imagedescription='description' image.jpeg

To convert a png to a jpeg in a new file:

    $ convert file.png file.jpeg

To convert a png to a jpeg *in place:*

    $ mogrify -format jpg file.png

To stack multiple images into a single image:

    $ convert file1.jpeg file2.jpeg -append result

To arrange multiple images horizonatally in a single image:

    $ convert file1.jpeg file2.jpeg +append result

## OpenDNS

Update /etc/default/ddclient

    run_daemon = "true"

Update /etc/ddclient.conf

    protocol=dyndns2
    use=web, web=myip.dnsomatic.com
    ssl=yes
    server=updates.opendns.com
    login=meredith.steve@gmail.com
    password='<opendns password>'
    opendns_network_label

To test

    sudo ddclient -verbose -file /etc/ddclient.conf

## CUPS Printing

* Admin via HTTP using port 631\.

## Crontab

To allow it to send mail, add to crontab file:

    MAILTO=<email>

Must install and configure ssmtp also: Edit /etc/ssmtp/ssmtp.conf:

    #
    # Config file for sSMTP sendmail
    #
    # The person who gets all mail for userids < 1000
    # Make this empty to disable rewriting.
    root=postmaster

    # The place where the mail goes. The actual machine name is required no
    # MX records are consulted. Commonly mailhosts are named mail.domain.com
    mailhub=email-smtp.us-east-1.amazonaws.com:587
    AuthUser=<user>
    AuthPass=<password>
    UseSTARTTLS=YES

    # Where will the mail seem to come from?
    #rewriteDomain=

    # The full hostname
    hostname=bitmine.org

    # Are users allowed to set their own From: address?
    # YES - Allow the user to specify their own From: address
    # NO - Use the system generated From: address
    FromLineOverride=NO

Add to /etc/ssmtp/revaliases:

    pi:<email>
    root:<email>

Install Sudo

    su
    apt-get install sudo
    adduser <user> sudo

## stdio

stderr

    foo 2> foo.txt

both

    foo &> foo.txt

