---
title: 命令备忘录：GDB
date: 2021-04-02 09:50:12
categories:
- 安装配置
- 应用软件
tags: 
- Linux
---

## 编译

`gcc Hello.cpp`  生成可执行文件a.out

`gcc -o Hello.o Hello.cpp`  生成可执行文件Hello.o

`gcc -g -o Hello.o Hello.cpp`  生成可调试的可执行文件Hello.o

## 调试

`gdb Hello.o`

## 常用GDB命令

`break` ：新增断点。后接一个参数，表示在指定位置增加断点，参数格式为：[源文件名:]<方法名> | [源文件名:]<行数>

`delete`：删除断点。后接零个参数，表示删除所有断点。后接一个参数，表示删除指定序号的断点，参数格式为：<序号>

`step`：往下执行语句，会进入函数。后接零个参数，表示往下执行一条语句。后接一个参数，表示往下执行指定数目的多条语句，参数格式为：<向下行数>

`next`：往下执行语句，不会进入函数。参数规范与step类似

`continue`：继续运行

`finish`：运行至当前函数返回后退出

`list`：查看代码

`frame`：查看帧栈

`backstrace`：查看整个调用栈

`info `：查看信息。后接一个参数，参数格式为：<args | locals>，args表示查看函数参数，locals表示查看局部变量

`print`：打印值。

