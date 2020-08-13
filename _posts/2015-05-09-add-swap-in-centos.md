---
title: Add Swap in CentOS
date: 2015-05-09T11:00:00+00:00
author: ZerUniverse
layout: post
categories: Linux
---

Today when I tried to compile an opensource software in my DigitalOcean VPS, I encountered a weird problem: every time when I `make` it, the command ends with error message `Killed`. However, I didn't press *CTRL+C* all the time. Then I keyed in command `top` and I saw `gcc` used up my memory! So I decided to add a swap file to enlarge my memory<!--more-->. Since DigitalOcean uses SSD for storage, swap won't make system "too slow".

First, check how much space that is still available in my server.

```bash
df -hl
```

Next, we add a file for swap.

```bash
dd if=/dev/zero of=/myswap bs=1024 count=1024k
```

This command means we copy 1024k blocks from `/dev/zero` to `/myswap`. Size of a single block is 1024 bytes. By the way, `/dev/zero` is a virtual device from where you can copy unlimited `0`.

Then we make that file as our swap file.

```bash
mkswap /myswap
swapon /myswap

#check
swapon -s
#you should see your swap file "myswap" here if you succeed

#let it automatically mounted when system starts
vim /etc/fstab
#in this file, append the following line at the end of the file
#/myswap          swap            swap    defaults        0 0

#give appropriate privilege
chown root:root /myswap
chmod 0600 /myswap
```

Now, the memory should be extended by 1G. And I successfully re-compiled the software.