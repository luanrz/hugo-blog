---
title: i3wm配置
date: 2020-09-04 16:21:00
categories:
- 安装配置
- 应用软件
tags: 
- Linux
---

i3wm是Linux下的一款窗口管理器(Windowns Manager), 通常简称为i3。 本文档基于Archlinux(其它发行版大同小异), 简单介绍了i3的常用软件与配置。

## 安装

```
sudo pacman -S i3-gaps polybar rofi feh picom
```

```
yay i3lock-fancy-rapid
```

| 序号 | 软件               | 功能                               |
| ---- | ------------------ | ---------------------------------- |
| 01   | i3-gaps            | i3-wm的增强版，支持间隙等特性      |
| 02   | polybar            | 状态栏，i3-bar的替代者             |
| 03   | rofi               | 启动菜单，dmenu的替代者            |
| 04   | feh                | 设置桌面壁纸                       |
| 05   | picom              | 透明支持                           |
| 06   | i3lock-fancy-rapid | 锁屏，i3lock的升级版，支持背景虚化 |


## 配置

### 配置polybar

1. 生成polybar配置文件

```
mkdir ~/.config/polybar
cp /usr/share/doc/polybar/config ~/.config/polybar/
```

2. 编辑polybar配置文件

```shell
vim ~/.config/polybar/config 
```

```
------------------------------------------
[bar/example]
...
border-size = 0
...
modules-left = i3 
modules-center = date
modules-right = memory cpu battery wlan eth
...

[module/wlan]
type = internal/network
interface = wlp0s20f3
...

[module/eth]
type = internal/network
interface = enp0s31f6
...
------------------------------------------
```

3. 新建polybar启动文件

```shell
vim ~/.config/polybar/launch.sh
```

```shell
killall -q polybar
while pgrep -u $UID -x polybar >/dev/null; do sleep 1; done
polybar example
```

```
chmod +x launch.sh
```

4. 启用polybar（参见[配置i3](#配置i3)）

```shell
vim ~/.config/i3/config
```

```shell
exec --no-startup-id ~/.config/polybar/launch.sh
```

### 配置rofi

1. 拷贝主题文件

```shell
mkdir ~/.config/rofi
cp -r /usr/share/rofi/themes ~/.config/rofi
```

2. 预览主题

```shell
rofi-theme-selector
```

3. 修改rofi配置

```shell
rofi -dump-config > ~/.config/rofi/config.rasi
vim ~/.config/rofi/config.rasi
```

```css
------------------------------------------
configuration {
    ...
    modi: "window,run,ssh,drun";
    show-icons: true;
    ...
}
....
------------------------------------------
```

4. 启用rofi（参见[配置i3](#配置i3)）

```shell
vim ~/.config/i3/config
```

```shell
## 绑定rofi：drun
bindsym --release $mod+space exec --no-startup-id rofi -show drun -theme ~/.config/rofi/themes/arthur.rasi
## 绑定rofi：window
bindsym $mod+Tab exec --no-startup-id rofi -show window -theme ~/.config/rofi/themes/sidebar.rasi
```

### 配置关停脚本

详情请参考[ArchLinux Wiki](https://wiki.archlinux.org/index.php/I3_(简体中文)#关机，重启和锁屏)

```
vim ~/i3exit.sh
```

```shell
------------------------------------------
#!/bin/sh
lock() {
    i3lock-fancy-rapid 5 3
}

case "$1" in
    lock)
        lock
        ;;
    logout)
        i3-msg exit
        ;;
    suspend)
        lock && systemctl suspend
        ;;
    hibernate)
        lock && systemctl hibernate
        ;;
    reboot)
        systemctl reboot
        ;;
    shutdown)
        systemctl poweroff
        ;;
    *)
        echo "Usage: $0 {lock|logout|suspend|hibernate|reboot|shutdown}"
        exit 2
esac

exit 0

------------------------------------------
```

```
chmod +x ~/i3exit.sh
```


### 配置i3

```shell
vim ~/.config/i3/config
```

```shell
------------------------------------------
## 将mod设置为Win键
set $mod Mod1

## 设置标题字体
font pango:monospace 10

# 关闭当前窗口（注释原有映射，与Mac保持一致）
#bindsym $mod+Shift+q kill
bindsym $mod+q kill

## 关闭底部常驻的i3bar（注释掉bar）
#bar {
#        status_command i3status
#}

## 设置窗口边框等等
new_window none
new_float normal
hide_edge_borders both
## 设置窗口间距
gaps inner 2
gaps outer 2

## 启动polybar
exec --no-startup-id ~/.config/polybar/launch.sh
## 设置背景图片
exec --no-startup-id feh --bg-fill ~/Pictures/156274745324.jpg
## 设置透明
exec --no-startup-id picom -b
## 启动中文输入法支持
exec --no-startup-id fcitx 
## Deepin系应用前置处理
exec --no-startup-id /usr/lib/gsd-xsettings &
## 将显示器DP2设置为主屏
exec --no-startup-id xrandr --output DP2 --primary

## 绑定rofi：drun
bindsym --release $mod+space exec --no-startup-id rofi -show drun -theme ~/.config/rofi/themes/arthur.rasi
## 绑定rofi：window
bindsym $mod+Tab exec --no-startup-id rofi -show window -theme ~/.config/rofi/themes/sidebar.rasi
## 绑定便签本：把当前窗口设为便笺本
bindsym $mod+Shift+minus move scratchpad
## 绑定便签本：呼出第一个便笺本
bindsym $mod+minus scratchpad show 
## 绑定锁屏功能
bindsym Mod4+l exec --no-startup-id /data/doc/synology/drive/Note/shell/bak/i3exit.sh lock
------------------------------------------
```
