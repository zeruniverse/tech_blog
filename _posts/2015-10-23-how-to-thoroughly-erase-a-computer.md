---
title: How to thoroughly erase a computer
date: 2015-10-23T12:00:00+00:00
author: ZerUniverse
layout: post
categories: Linux
---

Erasing a computer or a hard disk is essential sometimes such as selling your used computer or safely throwing your old hard drives<!--more-->.

In Windows, it's simple. Just Click the Windows Icon at the left-bottom corner and open "Settings" at the poped out menu. Then choose "Update & security" -> "Recovery" -> "Get started" -> "Remove everything" ... Then just wait for several hours until you get a new Windows. This step wipes everything on disks that you see in "This PC", and that said, it will not wipe out unallocated space or disk partitions not managed by Windows. Actually I'm not that convinced Windows perform a *thorogh erase* on the disks. To me it's more like a `rm -rf`. So I would use linux shred to perform a thorough clean up on a disk and reinstall Windows from USB disk or CD if necessary.

For Linux, if you don't want to keep the system, you can simply insert your USB drive which has bootable Linux ISO, then run:

```bash
#shred overwrite the destination with random data

shred -v /dev/sdX
#Wipe the entire hard drive
shred -v /dev/sdXY
#Wipe a partition
shred -v -u /dev/sdX
#Wipe a partition and delete all overwrited files
shred -v -f /dev/sdXY
#Force wipe
shred -v -f -u -n 10 /dev/sdXY
#Force wipe 10 times for partition /dev/sdXY and clear this disk.
#I think it's a good practice when selling your computer/hard disk.
```

If you want to keep the system and only erase your private data. It's enough to just shred your home directory. **DON'T USE rm**, it's very simple to [recover files deleted by rm command](/2015-05-10/recover-deleted-file-in-linux/). Just use shred:

```bash
#Delete a file
shred -u -v -n 10 example.file
#Delete a directory
find /path/to/directory -type f -execdir shred -v -n 10 {} \;
rm -rf /path/to/directory

#Delete home directory.
#First login as root
find /home/user -type f -execdir shred -v -n 10 {} \;
#Then delete the user together with its home folder
```