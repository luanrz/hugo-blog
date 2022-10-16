---
title: Termux配置
date: 2019-12-11 22:33:02
categories:
- 安装配置
- 应用软件
tags: 
- Linux
description: Termux是Android下的一款Linux环境, 本文简要介绍了Termux的部分实用配置
---

# Termux配置

## Termux初始配置

- 允许外部存储

    `termux-setup-storage`

- 修改termux镜像源
 
    `export EDITOR=vi && apt edit-sources`

        -----------------------------------------------------------
        http://mirrors.tuna.tsinghua.edu.cn/termux
        -----------------------------------------------------------

    `apt update && upgrade`

## 安装ubuntu

- 下载 ubuntu

    `pkg install wget  &&  wget https://raw.githubusercontent.com/Neo-Oli/termux-ubuntu/master/ubuntu.sh  &  bash ubuntu.sh`

- 修改dns
  
  `vi ~/ubuntu-fs/etc/resolv.conf`

        --------------------------------------
        nameserver 114.114.114.114
        --------------------------------------

- 修改ubuntu镜像源

    `vi ~/ubuntu-fs/etc/apt/source.list`

        --------------------------------------------------------------------------------------------------
        # 默认注释了源码仓库，如有需要可自行取消注释
        
        --------------------------------------------------------------------------------------------------

- 启动ubuntu

    `~/stdeb https://mirrors.ustc.edu.cn/ubuntu-ports/ xenial main restricted universe multiverse
        # deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ xenial main main restricted universe multiverse
        deb https://mirrors.ustc.edu.cn/ubuntu-ports/ xenial-updates main restricted universe multiverse
        # deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ xenial-updates main restricted universe multiverse
        deb https://mirrors.ustc.edu.cn/ubuntu-ports/ xenial-backports main restricted universe multiverse
        # deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ xenial-backports main restricted universe multiverse
        deb https://mirrors.ustc.edu.cn/ubuntu-ports/ xenial-security main restricted universe multiverse
        # deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ xenial-security main restricted universe multiverseart-ubuntu.sh`

    `apt update && upgrade`





