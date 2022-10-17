---
title: ArchLinux安装（一）
date: 2019-12-11 22:33:00
categories:
- 安装配置
- 操作系统
tags: 
- Linux
---

ArchLinux安装第一部分, 只包含基础环境的安装, 不包括桌面环境。本文档参考自ArchLinux官方Wiki的[Installation guide](https://wiki.archlinux.org/index.php/Installation_guide_(简体中文))。

## 一、硬盘分区

### 创建引导分区和根分区

`fdisk /dev/sda`	（进入第一个硬盘）

`g`	（如果是全新硬盘，使用g创建新的gpt分区表）

`n`	`回车`	`回车`	`+500M`	（创建新的分区作为引导分区，大小为500M）

`t`	`1`	（修改分区类型为EFI System）

`n`	`回车`	`回车`	`+10G`	（创建新的分区作为根分区，大小为10G）

`p`	（打印所有更改）

`w`	（保存所有更改）

###  格式化分区

`mkfs.fat -F32 /dev/sda1`	（格式化引导分区）

`mkfs.ext4 /dev/sda2`	（格式化根分区）

- 挂载分区

`mount /dev/sda2 /mnt`	（挂载根分区到airrootfs下的mnt目录）

`mkdir /mnt/boot`	（在airrootfs下的mnt目录创建boot子目录）

`mount /dev/sda2 /mnt/boot`	（挂载引导分区到airrootfs下的mnt/boot目录）
	
>（注意！！！chroot之前，所有的操作均是在内存中进行，airrootfs挂载为系统根目录，chroot在后面有介绍）

## 二、联网安装

### 联网

`wifi-menu`	（使用无线局域网）
	
`dhcp`	（使用自动拨号）

>（注意！使用手机开热点时，电脑可能不能正常解析DNS服务器，需要手动设置。）

`vim /etc/resolv.conf`

```
--------------------------------------------
nameserver 114.114.114.114）
--------------------------------------------
```

### 选择镜像源

`vim  /etc/pacman.d/mirrolist`	（修改镜像源）

```
--------------------------------------------------
Server = http://mirrors.neusoft.edu.cn/archlinux/$repo/os/$arch	 （选择镜像源）
--------------------------------------------------
```

>（大多数镜像源格式一致，只需要更改http与archlinux之间的内容即可）

### 安装基本包

`pacstrap /mnt base base-devel`
 
## 三、配置系统

### 设置分区自动挂载（生成fstab文件）

`genfstab -L /mnt >> /mnt/etc/fstab`	（自动生成fstab文件，如若按tab未正常补齐，说明前面安装错误）
	
`cat /mnt/etc/fstab`	（查看fstab文件是否生成成功，若内容不正确，会导致下次无法启动系统！）

### 更换根目录挂载点，系统操作权转移（Chroot）

`arch-chroot /mnt`	（将根目录挂载点由由airrootfs变为/dev/sda2（即airrootfs下的mnt目录））

>（注意！！！chroot之前，大部分操作是在内存中进行，chroot之后，一切操作均在硬盘上进行）	

### 趁现在有网，将必须的安装包下载下来

`pacman -S vim dialog wpa_supplicant	ntfs-3g `

`networkmanager`	（安装vim和网络相关的软件）

`pacman -S intel-ucode`	（安装intel相关驱动）

`pacman -S os-prober	grub efibootmgr`（安装引导相关软件）

### 设置Locale，主机名，Root密码

`vim /etc/locale.gen`	（去掉相关注释）

```
--------------------------------------------
en_US.UTF-8
zh_CN.UTF-8
zh_HK.UTF-8
zh_TW.UTF-8
--------------------------------------------
```

`locale-gen`	（使local生效）	

`vim /etc/locale.conf`	（编辑本地化文件）

```
--------------------------------------------
LANG=en_US.UTF-8	（设置默认本地本地化标准）
--------------------------------------------
```

`echo Archlinux > /etc/hostname`	（设置主机名）

`vim /etc/hosts`	（添加主机名对应信息，非必要）

`passwd`	（设置root密码）

### 设置时间

`timedatectl set-ntp true`	（更新系统时间）（在前面设置）

`ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`	（设置时区为上海）

`hwclock --systohc --utc`	（设置时间标准为UTC）

### 部署启动文件

`grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=arch`（在引导分区安装grub）

`vim /etc/lvm/lvm.conf` 

```
--------------------------------------------
use_lvmetad = 0（大约在40%处，将1改为0）
--------------------------------------------
```

`grub-mkconfig -o /boot/grub/grub.cfg`	（生成grub配置文件）

- 重启系统

`exit`

`reboot`

>至此，系统已经安装成功了！图形化界面安装参考“后续安装”
