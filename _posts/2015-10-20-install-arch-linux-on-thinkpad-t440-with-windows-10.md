---
title: Install Arch Linux on Thinkpad T440 with Windows 10
date: 2015-10-20T11:00:00+00:00
author: ZerUniverse
layout: post
categories: Linux
---

I usually use BIOS for installing Linux system. However, on this T440, a Windows is already installed in UEFI boot mode and I want to keep it. Thus, I decided to try installing Arch on UEFI mode. And actually, it turned out to be easier to use UEFI boot mode<!--more-->. You can see whether you've enabled UEFI mode on boot setting or simply type the command when you boot into your Arch iso.

```bash
efivar -l
```

And it's very easy to make an Arch bootable USB disk. Just [download it](https://www.archlinux.org/download/) on another computer and run the command:

```bash
dd bs=4M if=/path/to/archlinux.iso of=/dev/sdx && sync
```

with `sdx` to be your USB disk (e.g. `sdc`, **NOT sdcY**)

Let's start!

When you boot into your Arch iso, connect to wi-fi first using `wifi-menu`. Then partition your disk with `fdisk /dev/sdx`. type `d` to delete partition, type `n` to add partition. After that, formating the root partition using `ext4`.

```bash
mkfs.ext4 /dev/sdxY

#EFI partition can be formated with mkfs.vfat -F32 /dev/sdxY
```

The EFI partition is already generated by Windows with name "EFI system" in `fdisk -l`. So we just leave it there.

Now let's mount root and boot partition.

```bash
mount /dev/sdxY /mnt
#Assume /dev/sdxY is the root partition
mkdir -p /mnt/boot
mount /dev/sdxY /mnt/boot
#Assume /dev/sdxY is the boot partition. In UEFI, this is just the EFI partition
```

Then we can start installing the system, simply use:

```bash
pacstrap -i /mnt base base-devel iw wpa_supplicant dialog sudo
```

And then generate fstab:

```bash
genfstab -U -p /mnt >> /mnt/etc/fstab
```

Now we have the new system, arch-chroot into it to configure everything!

```bash
arch-chroot /mnt /bin/bash
vi /etc/locale.gen
# I uncommented en_US.UTF-8 UTF-8 zh_CN.UTF-8 UTF-8 zh_TW.UTF-8 UTF-8
locale-gen
echo LANG=en_US.UTF-8 &gt; /etc/locale.conf
ln -sf /usr/share/zoneinfo/America/Vancouver /etc/localtime

#Set time standard to be UTC
hwclock --systohc --utc
#For windows, need to edit registry in
#HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation\
#Add a REG_DWORD key RealTimeIsUniversal with value 1

passwd
#set password for root

bootctl install
#use bootctl for UEFI, if it's BIOS, grub is recommended

vi /boot/loader/entries/arch.conf
#add a boot entry
#title          Arch Linux
#linux          /vmlinuz-linux
#initrd         /initramfs-linux.img
#options        root=PARTUUID=xxxxxxxxxx rw
#
#You can get PARTUUID by running blkid /dev/sdxY

vi /boot/loader/loader.conf
#Edit loader config
#default  arch
#timeout  1
#editor   0

echo #myhostname &gt; /etc/hostname
vi /etc/hosts
#&lt;ip-address&gt; &lt;hostname.domain.org&gt; &lt;hostname&gt;
#127.0.0.1  localhost.localdomain   localhost   #myhostname
#::1        localhost.localdomain   localhost   #myhostname

reboot
#REMOVE YOUR USB DISK
```

Next, add a main user.

```bash
useradd -m -g users -G audio,video,floppy,network,rfkill,scanner,storage,optical,power,wheel,uucp -s /bin/bash YourUserName
passwd YourUserName
```

Make it a sudoer:

```bash
visudo
#After root ALL=(ALL) ALL
#Add YourUserName ALL=(ALL) NOPASSWD: ALL
```

Install Yaourt, a package management tool to manage packages from AUR:

```bash
vi /etc/pacman.conf
#Put the following into this conf file
#[archlinuxfr]
#Server = http://repo.archlinux.fr/$arch
#SigLevel = Optional TrustAll
pacman -S yaourt
```

Add finger print authencation support:

```bash
yaourt -S fprintd imagemagick
vi /etc/pam.d/system-local-login
#put the following line at the top of auth section
#auth      sufficient pam_fprintd.so
fprintd-enroll
#create your finger print signature
```

Now the new system should be dual-bootable.