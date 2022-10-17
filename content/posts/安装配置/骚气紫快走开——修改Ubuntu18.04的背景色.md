---
title: 骚气紫快走开——修改Ubuntu18.04的背景色
date: 2020-09-03 13:32:00
categories:
- 安装配置
- 操作系统
tags: 
- Linux
---

通过修改"GRUB界面","plymouth界面"以及"锁屏登陆界面", 来规避Ubuntu18.04的紫色背景

# GRUB界面

`sudo cp pictrue.jpg /boot/grub`

`cd /etc/default/`

`sudo cp grub grub.bak`

`sudo vim grub`

```
GRUB_GFXMODE=1366x768
```

`sudo update-grub`

# plymouth界面

`cd /usr/share/plymouth/themes/ubuntu-logo`

`sudo cp ubuntu-logo.script ubuntu-logo.script.bak` 

`sudo vim ubuntu-logo.script`

```
Window.SetBackgroundTopColor (0.00, 0.00, 0.00);
Window.SetBackgroundBottomColor (0.00, 0.00, 0.00);
```

# 锁屏登陆界面

`cd /usr/share/gnome-shell/theme/`

`sudo cp ubuntu.css ubuntu.css.bak`

`sudo vim ubuntu.css`

```
#lockDialogGroup {
  background: black;
}
```
