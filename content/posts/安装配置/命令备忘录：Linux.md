---
title: 命令备忘录：Linux
date: 2021-04-02 09:50:12
categories:
- 安装配置
- 应用软件
tags: 
- Linux
---

## 会话管理——使用`screen`恢复掉线的shell会话

`screen -S myScreen`

`screen -r myScreen`

## 显示当前目录所占用的磁盘空间

`du -h --max-depth=1`

## 访问NFS服务

`sudo pacman -S core/nfs-utils` #ArchLinux下安装nfs-utils，不然没有showmount命令

`showmount -e 192.168.1.105` #显示指定IP上挂载的卷

`sudo mount -t nfs -o proto=tcp,nolock 192.168.1.105:/volume1/homes homes/` #开始挂载

## 访问SMB服务

`sudo pacman -S smbclient` #安装

` smbclient -L 192.168.0.100 -U username%password` # 显示共享目录

` smbclient //192.168.0.100/directory -U username%password` # 进入上述共享目录中的directory目录

## 一键删除所有项目下的target

`find . -name target | awk '{print "rm -rf "$1}' | sh`

## 扫描局域网所有IP与端口

`nmap -sP 192.168.1.0/24`

`nmap -A 192.168.1.100`

## 压缩与解压

|        |             压缩             |         解压          |
| :----: | :--------------------------: | :-------------------: |
| tar.gz | tar czvf  test.tar.gz  test/ | tar xzvf  test.tar.gz |
| tar.xz | tar cJvf  test.tar.xz test/  | tar xvf  test.tar.xz  |
|  zip   |     zip test.zip test/*      |    unzip test.zip     |
|  rar   |     rar test.rar test/*      | (un)rar x(e) test.rar |

