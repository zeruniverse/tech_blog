---
title: Recover Deleted File in Linux
date: 2015-05-10T11:00:00+00:00
author: ZerUniverse
layout: post
categories: Linux
---

I still clearly remembered once I mistyped `mv` as `rm` and I deleted my assignment which takes me an entire weekend to write. That time, I wanted to execute the command: `mv xxx.c src/xxx.c` (I wanted to put all source code in `src` folder) but what I really typed in the terminal was: `rm xxx.c src/xxx.c`. Right after I pressed the enter key, I realized that was a disaster. But it was already too late (though `src/xxx.c` doesn't exist, `xxx.c` will be deleted anyway if that command executes). Fortunately, I recovered it finally but it took much more time than that of recovering a file in Microsoft Windows<!--more-->.

When you notice you mistakenly deleted a file, the first step to take is always **DISABLING WRITE** to that disk **IMMEDIATELY**. `Delete` command won't erase your data from your disk but **WRITE COMMAND WILL**. Here, I use my system as example. I use ArchLinux as OS and ext4 as file system. All my personal data is stored under `/home/jeffery` and I mount `/dev/sdb4` on `/home`. But home directory for my root user is `/root`. So I first use root user to make the disk *read only*. (For next few steps, I use user `root`. Since home directories of other users are in `/home` and that's read-only now)

```bash
su root
#password
fuser -km /home #kill all things currently using /home
umount /home
mount –t ext4 –o ro /dev/sdb4 /home #ro = read only
```

Then, I use a tool called undelete to help me.
```bash
wget http://downloads.sourceforge.net/project/extundelete/extundelete/0.2.4/extundelete-0.2.4.tar.bz2?r=&amp;ts=1431286176&amp;use_mirror=tcpdiag
tar jxvf extundelete-0.2.4.tar.bz2
cd extundelte-0.2.4
./configure
make
make install
```

The next step is using undelete to recover the file. Now, assume we mistakenly deleted file `a.temp` in `/home/jeffery/test/`.

First we need to have the id of the parent folder. Then, we should unmount that disk and restore the file.

```bash
ls -id /home/jeffery/test #result = 2892570
umount /dev/sdb4
extundelete /dev/sdb4 --inode 2892570  # find inode for a.temp is 2892625
extundelete /dev/sdb4 --restore-inode 2892625
```

The result for `extundelete /dev/sdb4 --inode 2892570` is in the following figure.

<a href="/postimg/extundelete.png"><img src="/postimg/extundelete.png" alt="extundelete" width="100%" /></a>

Finally, we can find the restored file in `/RECOVERED_FILES`. File name for the restored file is `file.2892625`. It's the exact file I deleted just now.

The tool `undelete` can also help restore the whole directory you delete or simply restore everything.