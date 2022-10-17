---
title: ArchLinux安装（二）
date: 2019-12-11 22:33:01
categories:
- 安装配置
- 操作系统
tags: 
- Linux
---
 
 ArchLinux安装第二部分, 包括图形化界面与实用软件的安装及实用配置。本文档参考自ArchLinux官方Wiki的[General recommendations](https://wiki.archlinux.org/index.php/General_recommendations_(简体中文))。

## 四、后续安装

### 安装图形化界面

`pacman -S xf86-video-intel`	（安装显卡驱动）

`pacman -S xorg xfce4 xfce4-goodies sddm`	（安装桌面桌面环境与桌面管理器）

`pacman -S network-manager-applet`	(安装NetworkManager插件)

`systemctl enable sddm`	（开机自动启动sddm）

`systemctl enable NetworkManager`	（开机自动启动NetworkManager）

`systemctl disable netctl`	（取消开机自动启动netctl）

>netctl和networkmanager互斥！


### 新建用户并赋予其sudo权限

`useradd -m -G wheel luanrzh`	（创建用户luanrzh）

`passwd luanrzh`	（修改luanrzh密码）
	
`pacman -S sudo`（安装sudo）

`vim /etc/sudoers`	（配置sudo）

### 创建交换文件

`fallocate -l 4G /swapfile`

`chmod 600 /swapfile`

` mkswap /swapfile`
	
`swapon /swapfile`

`vim /etc/fstab`

### 安装yaourt

`vim /etc/pacman.conf`

```	
------------------------------
[archlinuxcn]
SigLevel = Optional TrustAll
Server = http://repo.archlinuxcn.org/$arch
------------------------------
```

`pacman -Syu yaourt`

### 安装中文环境

`pacman -S noto-fonts-cjk`	（谷歌出品的noto字体，大而全）

//`pacman -S wqy-microhei`	（文泉驿微米黑字体，小而精，只支持中文）

> 字体文件夹：/usr/share/fonts/（全局） ~/.fonts（用户，已过时）

`pacman -S  fcitx-sogoupinyin  fcitx-im fcitx-configtool`	（fcitx输入法）

`vim .xprofile`	（配置fcitx输入法）

```
------------------------------
export XMODIFIERS=@im=fcitx
export QT_IM_MODULE=fcitx
export GTK_IM_MODULE=fcitx
------------------------------
```

### 安装实用软件

`yaourt google-chrome`（谷歌浏览器）

`yaourt nutstore`	（坚果云）

`yaourt wps-office`	（WPS Ofiice）
  
>字体问题：安装完WPS后坚果云的界面字体变得贼丑，原因是wps新增的字体优先级高于原先的默认字体。
[调整字体优先级](https://www.copie.cn/index.php/archives/archlinux-%E8%B0%83%E6%95%B4%E5%AD%97%E4%BD%93%E4%BC%98%E5%85%88%E7%BA%A7-1.html "archlinux 调整字体优先级")

`yaourt netease-cloud-music`	(网易云音乐)

`yaourt visual-studi-code` 	（VS Code）

`yaourt android-studio`	（Android Studio）

`yaourt Moeditor`	（一款markdown编辑器）

`pacman -S yakuake guake`	（两款终端模拟器，可配合快捷键实现快速下拉）

`pacman -S virtualbox`	(virtualbox虚拟机)

`pacman -S linux-headers`	（不安装这个virtualbox虚拟机可能会报错）

`pacman -S baobab`（gnome中的磁盘使用情况分析工具）

`pacman -S filezilla`（ftp客户端）

`pacman -S remmina`（远程桌面连接工具）

`yaourt remmina-plugin-rdesktop`	（remmina RDP插件）

`pacman -S File-Roller  XArchive`（两款解压缩工具）

`pacman -S alacarte`	(应用程序主菜单编辑软件)
	
`pacman -S nmap`	(局域网扫描工具)

`yaourt wechat_web_devtools`	(微信开发者工具)

`pacman -S dia umbrello`	(两款UML绘图工具)

`pacman -S gimp`	(画图软件,PS替代品)

`pacman -S kolourpaint`	(另一款画图软件)

`yaourt eclim`	（Vim下的Eclipse,并没有省内存,没卵用）

`yaourt i-nex`	(系统信息查看软件)
  
`yaourt dbeaver`	（数据库客户端）
  
`yaourt calibre`	（电子书）
	
### 界面美化

`yaourt papirus-icon-theme`（图标）

`yaourt numix-icon-theme`（图标）

`yaourt numix-circle-icon-theme`（圆形图标）

`yaourt arc-gtk-theme`（样式，主题）

`pacman -S docky`（底部出现横杠，点击设置->窗口管理器微调->合成器，取消“在dock窗口下显示阴影”）

### 使用MTP协议与Android手机传输数据

`yaourt mtpfs`（MTP支持）

`gvfs-mtp `（GNOME Files文件管理器集成）

`vim/etc/fuse.conf `

`mtpfs -o allow_other ~/mnt`

### 安装deepin下的国产windows软件

`vim /etc/pacman.conf` （开启multilib库，去掉#注释）

```
------------------------------
[multilib]
Include = /etc/pacman.d/mirrorlist
------------------------------
```

`yaourt -Sy deepin.com.qq.im`	（QQ）

`yaourt -Sy deepin.com.qq.office`	（Tim）

`yaourt -Sy deepin.com.thunderspeed `	（迅雷）

`yaourt -Sy deepin-wine-thunderspeed`	（迅雷备选方案）

`yaourt -Sy deepin-baidu-pan`	（百度云）


### 安装LAMP

1. #### apache与php

`pacman -S php-apache`	（安装apache、php、php扩展:libphp）

`mousepad /etc/httpd/conf/httpd.conf`	（配置apache——php扩展）

> 参见 [Archlinux wiki](https://wiki.archlinux.org/index.php/Apache_HTTP_Server#PHP)

```
------------------------------
#LoadModule mpm_event_module modules/mod_mpm_event.so（注释）
LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
（去掉注释）
.
LoadModule php7_module modules/libphp7.so
AddHandler php7-script php	（将这一行放在LoadModule 的末尾）
.
Include conf/extra/php7_module.conf	（将这一行放到Include列表的末尾）
------------------------------
```

> 使用命令：“php -S localhost:8000 -t public_html/ ”可以独立运行PHP

2. mysql

`pacman -S mariadb`	（安装mysql数据库）
	
`mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql`	（mysql目录配置）

`systemctl start mysqld`	（启动数据库）

`mysql_secure_installation`	（mysql安全配置）

`pacman -S  dbeaver`	（安装数据库管理工具）

3. phpmyadmin

`pacman -S  phpmyadmin`	（安装phpmyadmin）

`mousepad /etc/httpd/conf/extra/phpmyadmin.conf`	（创建Apache配置文件）

```
------------------------------
Alias /phpmyadmin "/usr/share/webapps/phpMyAdmin"
<Directory "/usr/share/webapps/phpMyAdmin">
	DirectoryIndex index.php
	AllowOverride All
	Options FollowSymlinks
	Require all granted
</Directory>
------------------------------
```

`mousepad /etc/httpd/conf/httpd.conf`	（将上个文件的引用加入到httpd.conf）

```
------------------------------
# phpMyAdmin configuration
Include conf/extra/phpmyadmin.conf（将这一行放到Include列表的末尾）
------------------------------
```

//`mousepad /etc/webapps/phpmyadmin/config.inc.php`	（配置phpmyadmin）

`mousepad /etc/php/php.ini`（配置mysqli 扩展）

```
------------------------------
extension=mysqli（去掉;注释）
------------------------------	
```

> 不执行上述操作的话，在进入phpmyadmin时会显示"缺少 mysqli 扩展“

### 安装LNMP

参见[Arch Linux服务器安装LNMP (Nginx, MariaDB, PHP7)](https://www.linuxdashen.com/install-lemp-nginx-mariadb-php7-arch-linux-server)

## 五、后续配置

### 设置声卡（解决网易云音乐没有声音的问题）

`pacman -S alsa-utils`
	
`alsamixer`

`F6`	（切换至HDA Intel PCH，将Master调到100）

`vim ~/.asoundrc`	（设置用户级默认声卡，系统级修改/etc/asound.conf ）

```
------------------------------
pcm.!default {
	type hw
	card PCH
}
ctl.!default {
	type hw           
	card PCH
}
------------------------------
```

### 设置开机不输入密码直接进入界面

`sudo vim /etc/sddm.conf`

```
------------------------------
[Autologin]
User=luanrzh
Session=xfce
------------------------------
```

### 取消延迟启动grub

`sudo vim /boot/grub/grub.cfg`

```
------------------------------
set timeout=0	（将5改为0）
------------------------------
```


### 解决TIM(QQ)无法接受图片的问题
    
`sudo vim /etc/sysctl.conf`
    
```
------------------------------
#disable ipv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.<enp9s0>.disable_ipv6 = 1
net.ipv6.conf.<lo>.disable_ipv6 = 1
------------------------------
```
  
`sudo sysctl -p`
  
`rm rf ~/.deepinwine/Deepin-TIM/`
  
### 关闭蜂鸣器beep

- 临时关闭
    
`sudo rmmod pcspkr`
    
- 永久关闭
    
`sudo vim /etc/modprobe.d/nobeep.conf`
    
```
--------------------
blacklist pcspkr
--------------------
```
    
    
### 设置锁屏

- 安装i3lock

`yay i3lock-fancy-rapid-git`

- 让xflock4调用i3lock

`xfconf-query -c xfce4-session -p /general/LockCommand -s "i3lock-fancy-rapid 5 3" --create -t string`
