# NOOBIES-GUIDE-TO-ARCH
There seems to be a misconception about the difficulty of using Arch Linux for beginner users. While Arch Linux should not be your first choice when migrating from Windows or OSX to Linux, it is by no mean unfeasable.
For the beginner linux user who has had experience with one or two distros, it may be time to play around with a minimalist distro. For those people, Arch Linux is the perfect choice. The installation seems daunting, but once you do it a few times, you'll realize it really is not.

*In this guide, I will be explaining:*
* How to install Arch Linux
* How to configure your own Desktop Environment
* How to use pacman

This guide is in no way meant to replace the Arch Wiki's installation instructions for Arch Linux. This is merely a summary, and may not work for everyones hardware. If you plan on doing an installation for specific hardware, please refer to: https://wiki.archlinux.org/index.php/Installation_guide

*DISCLAIMER*: It is recommended to follow my guide within a virtual machine, so you can learn the basics. This guide can be fully adapted to hardware if you so please, but do so at your own risk!

# Installation
Before we can learn Arch Linux, we need to have Arch Linux. An experienced user can install Arch Linux in under 3 minutes, but it is important to take your time when you are first learning. I will be covering the steps on installation to get your system up and running, but there may be some additional steps you would like to follow from the Arch Wiki. With that being said, let's get started.

First, we need to download the Arch Linux installation iso, found at: https://www.archlinux.org/download/
Burn the iso onto a CD, or a USB flash drive, and use it as your boot device.

We should now be in the installation live CD and we can get started.

This guide will only be covering systems using EFI, if you are not sure if you have EFI, type `ls /sys/firmware/efi/efivars` If you recieve no output, you are not booted into EFI, and should consult the Arch Wiki for the rest of this installation.

Assuming you already have internet connection (`ping archlinux.org` if you are unsure), we can partition our disks.

A basic partition table includes a:
boot partition: 512M
root partition: Remainder of drive

type `lsblk` to list your block devices.
Find the disk you wish to install Arch Linux onto

In my case it is vda (because I am using a virtual machine).
we are going to type `fdisk /dev/<your drive>` (vda/sda)

We want to type `g` and press enter, to create a gpt partition table
And then press `n` to create a new partition
Choose partition number `1`
And press enter for First Sector
For Last Sector, input `+512M`
now we need to change the type of our partition from LFS to EFI

To do this we will type `t`, and select 1
Now we have successfully created our boot partition

To create our root partition, we will create a new partition `n`
Select partition number `2`
Press `enter` for default First sector
Press `enter` for last sector as well
and now, type `w` to write the changes to your disk

Typing `lsblk` will now show us two partitions branching from the disk we partitioned.

Now we need to create the filesystems:

For the boot partition, we will type `mkfs.fat -F 32 /dev/sda1` <- whatever disk you partitioned
For the root partition, we will type `mkfs.ext4 /dev/sda2` <- whatver disk you partitioned

Now it's time to mount our root partition

`mount /dev/sdX2 /mnt` <-replace X with drive identifier

Now we can install the base system with:
`pacstrap /mnt base linux linux-firmware`

Now we can generate an fstab:
`genfstab -U /mnt >> /mnt/etc/fstab`

Time to chroot and complete the installation:
`arch-chroot /mnt`

To generate timezone and set time:
`ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`
`hwclock --systohc`

At this point you can download a text editor with `pacman -S vim` <- replace vim with your favorite editor

Edit /etc/hostname:
`hostname` <- this is how you name your computer

Edit /etc/hosts
```
127.0.0.1 localhost
::1       localhost
127.0.1.1 <hostname>
```
is the way I configure hosts

Generate an initramfs with:
`mkinitcpio -P`

and finally set your password with:
`passwd`

## Setting up a desktop environment
In this section we will set up:
* A Bootloader
* Network Manager
* A user
* A desktop environment
* Sudo

To get started with a bootloader, we will choose grub

run `pacman -S grub efibootmgr` to install grub

Now we need to make our boot directory:
`mkdir /boot/efi`
and mount our partition to it
`mount /dev/sda1 /boot/efi`

To install grub:
`grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB`

and then `grub-mkconfig -o /boot/grub/grub.cfg`

Finally, before we reboot, we need to set up network manager:
`pacman -S networkmanager`

To enable it, `systemctl enable NetworkManager`

Now `exit` chroot, and `reboot`.

Login as `root` and with the password you set earlier

Before we set up a Desktop Environment, lets make a user
`useradd -m <username>`
And let's add our new user to some groups
`usermod -a -G wheel,storage,users <user>`
Finally let's set a password:
`passwd <user>`

The guide will cover KDE Plasma as our Desktop Environment as choice:
`pacman -S plasma-meta` to install plasma
`pacman -S sddm` to install our display manager

install a terminal for the next step
`pacman -S konsole`

To start up plasma, we need to start our display manger with
`systemctl enable sddm`
`systemctl start sddm`

Now login as the user we just created

Open your terminal, and type `su` to login as your root user

## sudo
To make use of our root user while logged in as a standard user, we need sudo.
`pacman -S sudo` to install

Before our user can use sudo, we must add them to the /etc/sudoers file

beneath
`root ALL=(ALL) ALL` add
`<username> ALL=(ALL) ALL`
Save the file, and you can now `exit` your root user

From now on, any command you would like to execute as root, can be performed with `sudo` as a prefix.

Congrats, you now have arch linux set up and ready to use. That wasn't so hard was it?
