---
title: Install dwm - dynamic window manager with Chinese environment
date: 2015-10-20T12:00:00+00:00
author: ZerUniverse
layout: post
categories: Linux
---

DWM is a simple wondow manager. Since I only need terminal and browser in most cases. Dwm would be a better choice for me than Gnome<!--more-->.

The first step is installing drivers and `Xorg`. Before that, make sure `multilib` in `/etc/pacman.conf` is uncommneted.

```bash
sudo pacman -S lib32-mesa xorg-server xorg-server-utils xorg-apps xorg-xinit
```

Since I want the dwm to support Chinese, I installed `dwm-pango` from `yaourt`. Otherwise, the Chinese character might display incorrectly on the status bar. Some fonts are also needed to display Chinese character correctly. I like the default setting by dwm so I didn't change `config.h`. Remember to recompile if you change `config.h` during your set up.

```bash
yaourt dwm-pango dmenu adobe-source-han-sans-cn-fonts adobe-source-han-sans-tw-fonts wqy-microhei wqy-zenhei wqy-bitmapfont ttf-arphic-ukai ttf-arphic-uming opendesktop-fonts otf-texgyre
```

Then install some apps such as xterm and fcitx (Chinese Input Method)

```bash
yaourt firefox fcitx-im fcitx-sunpinyin fcitx-configtool xterm feh fluxgui wmname
```

Now write config files.

First, we write `.xinitrc` as below:

```bash
xrdb ~/.Xresources
export LANG=zh_CN.UTF-8
export LC_ALL="en_US.UTF-8"
export XMODIFIERS='@im=fcitx'
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
exec fcitx &gt; /dev/null &amp;

while true
do
   if acpi -a | grep off-line &gt; /dev/null; then
       xsetroot -name "Bat. $( acpi -b | awk '{ print $4 }' | sed 'N;s/\n/ /g' ) | Vol. $(amixer get Master | tail -1 | awk '{ print $5}' | tr -d '[]') | $(date +"%a, %b %d %T")"
   else
       xsetroot -name "Vol. $(amixer get Master | tail -1 | awk '{ print $5}' | tr -d '[]') | $(date +"%a, %b %d %T")"
   fi
   sleep 5s
done &amp;

# automatically change monitor color tempreture
exec xflux -l 49 -g -123 &gt;/dev/null&amp;

wmname LG3D

# set background to .bg.jpg
feh --bg-scale ~/.bg.jpg

exec dwm
```

Then we write `.Xresources` to configure Uxterm

```text
!!!!!!!!!!!!!!!!!!!!!
!! UXTerm settings !!
!!!!!!!!!!!!!!!!!!!!!
UXTerm*background: #000000
UXTerm*foreground: green
UXTerm*customization: -color
UXTerm*toolBar: false
UXTerm*highlightSelection: true
UXTerm*VT100*underLine: on
UXTerm*VT100*colorMode: on
UXTerm*VT100*dynamicColors: on
UXTerm*VT100*colorAttrMode: off
UXTerm*VT100*colorBDMode: on
UXTerm*VT100*colorBD: white
UXTerm*VT100*colorULMode: on
UXTerm*VT100*colorUL: red
UXTerm*VT100*titeInhibit: true

UXTerm.VT100*color0: black
UXTerm.VT100*color1: red3
UXTerm.VT100*color2: green3
UXTerm.VT100*color3: yellow3
UXTerm.VT100*color4: blue2
UXTerm.VT100*color5: magenta3
UXTerm.VT100*color6: cyan3
UXTerm.VT100*color7: gray90
UXTerm.VT100*color8: gray30
UXTerm.VT100*color9: red
UXTerm.VT100*color10: white
UXTerm.VT100*color11: yellow
UXTerm.VT100*color12: blue
UXTerm.VT100*color13: magenta
UXTerm.VT100*color14: cyan
UXTerm.VT100*color15: white
UXTerm.VT100*cursorColor: lime green
UXTerm*faceName:  WenQuanYi Micro Hei Mono:style=Regular
UXTerm*faceSize: 9
```

Write `.bash_profile` to start dwm automatically after login:

```bash
#
# ~/.bash_profile
#

[[ -f ~/.bashrc ]] &amp;&amp; . ~/.bashrc

[[ -z $DISPLAY &amp;&amp; $XDG_VTNR -eq 1  ]] &amp;&amp; exec startx
```

Then, `startx` and use `dmenu` to call `fcitx-configtool`. You can configure `fcitx` here.

The DWM should be ready. A full list of hot keys:

```text
[Alt]+[p]                  dmenu for running programs like the x-www-browser
[Shift]+[Alt]+[q]          To quite dwm cleanly
[Shift]+[Alt]+[c]          To kill the focused window
[Shift]+[Alt]+[Enter]      Launch terminal
[Shift]+[Alt]+[1,2,3,...]  Move current window to area 1,2,3...
[Alt]+[1,2,3,...]          Switch to area 1,2,3...
[Alt]+tab                  Switch view area
[Alt]+j                    Switch to next window
[Alt]+k                    Switch to previous window
[Alt]+b                    Hide/show status bar
[Alt]+t [Alt]+f [Alt]+m    Swith the layout
[Alt]+space                Switch the front and background window
[Alt]+[Enter]              Switch the window position
```