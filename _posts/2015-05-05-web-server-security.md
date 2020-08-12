---
title: Web Server Security
date: 2015-05-05T11:00:00+00:00
author: ZerUniverse
layout: post
categories: Linux
---

The security configuration of a web server is very important especially when some confidential data is stored in its database. However, I didn’t do anything to ensure the safety of my server after I installed operating system into it. So I decide to make up this today<!--more-->.

I use *CentOS 7 (x64)* as my operating system. Steps I take includes changing ssh port, forbidden ping, Configure iptables, change user privillege, disallow any change of some important file and etc.

1. Change the ssh port

Though a strong password might be enough to protect the server, it would be better if potential attackers don’t know you have ssh installed. Thus, changing the ssh port would be a good idea.

```bash
vim /etc/ssh/sshd_config

#Find [#Port 22], delete the number key and modify 22 to the port you choose

service sshd restart
```

2. Disable system service you don’t need

There might be vulnerability in some system services, thus disabling services you don’t need can enhance the security of your server.

```bash
service acpid stop chkconfig acpid off #stop battery management service
service autofs stop chkconfig autofs off #stop autofs
service bluetooth stop chkconfig bluetooth off #stop bluetooth
service cups stop chkconfig cups off #stop printing system
service cpuspeed stop chkconfig cpuspeed off #stop cpu speed monitor

#If you don't use IPv6
service ip6tables stop chkconfig ip6tables off #stop IPv6

#If you want to restart some service you disable:
#service ip6tables start chkconfig ip6tables on
```

If you don’t want to use IPv6, also do the followings:

```bash
vim /etc/sysconfig/network
#add the following line to this file:
#NETWORKING_IPV6=no

vim /etc/sysconfig/network-scripts/ifcfg-eth0
#add the following two lines to this file:
#IPV6INIT=no
#IPV6_AUTOCONF=no
```

Then create a new file under `/etc/modprobe.d/`, Assume it’s `stopipv6.conf`

```bash
vim /etc/modprobe.d/stopipv6.conf
```

In this file, add two lines:

```
alias net-pf-10 off
options ipv6 disable=1
```

3. Disallow users you don’t need

Edit `/etc/passwd`, comment the user you don’t need (do not delete them in case you need them in future)

```
#For example, if adm is not needed, put a number key at first of that line.
#This is my file after commenting useless users.


root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
#adm:x:3:4:adm:/var/adm:/sbin/nologin
#lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
#shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
#halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
#operator:x:11:0:operator:/root:/sbin/nologin
#games:x:12:100:games:/usr/games:/sbin/nologin
#ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:999:998:User for polkitd:/:/sbin/nologin
avahi:x:70:70:Avahi mDNS/DNS-SD Stack:/var/run/avahi-daemon:/sbin/nologin
avahi-autoipd:x:170:170:Avahi IPv4LL Stack:/var/lib/avahi-autoipd:/sbin/nologin
#postfix:x:89:89::/var/spool/postfix:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
```

4. Disallow groups you don’t need

Edit `/etc/group`, comment the group you don't need (do not delete them in case you need them in future)

```
#For example, if adm is not needed, put a pound at first of that line.

#adm:x:4:
#lp:x:7:
#games:x:20:
#dip:x:40:
```

5. Install `sudo` and only allow certain user to login via `ssh`

```bash
yum install sudo
adduser xxx
passwd xxx
chmod u+w /etc/sudoers
vim /etc/sudoers

#find [root ALL=(ALL) ALL]
#after that add [xxx ALL=(ALL) ALL]

chmod u-w /etc/sudoers
```

Then, only allow *xxx* to login via `ssh`

```bash
vim /etc/pam.d/sshd

#insert the following into the first line
#auth required pam_listfile.so item=user sense=allow file=/etc/sshallowusers onerr=fail

vim /etc/sshallowusers

#add the following one line to the new file:
#xxx
```

6. Avoid IP spoofing

Edit `/etc/host.conf`, make it the same as following:
```
order hosts,bind
multi on
nospoof on
```

7. Disallow FTP user to login and limit the folder user can get access via FTP

Edit `/etc/host.conf`, make it the same as following:

```bash
usermod -s /sbin/nologin xxxftp
vim /etc/vsftpd/vsftpd.conf
#chroot_list_enable=YES //limit the folder user can get access via FTP
# (default follows)
#chroot_list_file=/etc/vsftpd/vsftpd.chroot_list
#Then, edit /etc/vsftpd/vsftpd.chroot_list

/etc/init.d/vsftpd restart
```

8. IP tables configuration

```bash
iptables -F
iptables -X
iptables -P FORWARD ACCEPT
iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT

/sbin/iptables -A INPUT -i lo -j ACCEPT #local address pass
/sbin/iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT #allow exist connection
/sbin/iptables -A INPUT -p tcp --dport XXXX -j ACCEPT #ssh (YOUR SSH PORT)
/sbin/iptables -A INPUT -p tcp --dport 80 -j ACCEPT #http
/sbin/iptables -A INPUT -p tcp --dport 443 -j ACCEPT #https
/sbin/iptables -A INPUT -p tcp --dport 20 -j ACCEPT #ftp
/sbin/iptables -A INPUT -p tcp --dport 21 -j ACCEPT #ftp
/sbin/iptables -A INPUT -p udp --sport 53  -j ACCEPT #DNS
/sbin/iptables -A INPUT -p udp --sport 123 -j ACCEPT #ntp
/sbin/iptables -A INPUT -p icmp --icmp-type echo-request -j DROP   #DISALLOW ping/ACCEPT if you want to allow PING
/sbin/iptables -P INPUT DROP

/sbin/iptables -A OUTPUT -o lo -j ACCEPT #local address pass
/sbin/iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT #allow exist connection
/sbin/iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT #http
/sbin/iptables -A OUTPUT -p tcp --dport 443 -j ACCEPT #https
/sbin/iptables -A OUTPUT -p udp --dport 53  -j ACCEPT #DNS
/sbin/iptables -A OUTPUT -p udp --dport 123 -j ACCEPT #ntp
/sbin/iptables -A OUTPUT -p icmp --icmp-type echo-request -j DROP #DISALLOW ping others/ACCEPT if you want to allow PING
/sbin/iptables -P OUTPUT DROP

/sbin/service iptables save
iptables -vnL
```

9. Disallow adding new user and change to system file

```bash
chattr +i /etc/passwd
chattr +i /etc/shadow
chattr +i /etc/group
chattr +i /etc/gshadow
chattr +i /etc/services

#If you want to recover (e.g. add new user) in the future, do the following:
#chattr -i /etc/passwd
#chattr -i /etc/shadow
#chattr -i /etc/group
#chattr -i /etc/gshadow
#chattr -i /etc/services
```