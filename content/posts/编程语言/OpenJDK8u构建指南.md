---
title: OpenJDK8u构建指南
date: 2020-01-20 17:42:00
categories:
- 编程语言
- Java
tags:
- Java
- JVM
---

官方OpenJdk8u构建指南个人翻译版本。官方地址为[README-builds.html](http://hg.openjdk.java.net/jdk8u/jdk8u/file/120809c21ad7/README-builds.html)

# 介绍

这个README文件包含了OpenJDK的构建说明。构建OpenJDK的源码需要一定的专业技术知识。

!!!!!!!!!!!!!!! 这是对这份文档的一次重大改写!!!!!!!!!!!!!!! 

概述如下：
- 当前构建方式是“`configure && make`”
- GNU make的版本应当大于或等于3.81
- 构建是可伸缩的，比如：可以使用更多的处理器来完成构建过程，以减少构建时间
- 嵌套与递归的make调用已经明显减少，fork/exec 与子进程生成的总量也相应减少
- 不再支持 Windows MKS
- Windows Vistual Studio 的 `vsvars*.bat` 和 `vcvars*.bat` 等文件会自动运行
- 构建OpenJDK时，不再使用Ant
- 不再支持在配置构建过程时使用 ALT_* 环境变量

# 目录
- [介绍](#介绍)
- [使用Mercurial](#使用Mercurial)
    - [获取源码](#获取源码)
    - [仓库结构](#仓库结构)
    - [源码规范](#源码规范)
- [构建](#构建)
    - [系统设置](#系统设置)
        - [Linux](#linux)
        - [Solaris](#solaris)
        - [MacOSX](#macOSX)
        - [Windows](#windows)
    - [Configure](#configure)
    - [Make](#make)
- [测试](#测试)

- [附录A：提示和技巧](#附录a提示和技巧)
    - [常见问题解答](#常见问题解答)
    - [构建性能技巧](#构建性能技巧)
    - [故障排查](#故障排查)
- [附录B：GNU-Make信息](#附录bgnu-make信息)
- [附录C：构建环境](#附录c构建环境)

# 使用Mercurial
OpenJDK源码由版本控制系统 [Mercurial](http://mercurial.selenic.com/wiki/Mercurial) 维护，如果你还不熟悉 Mercurial ，请参阅 [Beginner Guides](http://mercurial.selenic.com/wiki/BeginnersGuides) 或参考 [Mercurial Book](http://hgbook.red-bean.com/) 。本书的前几章对“Mercurial是什么“以及“Mercurial如何工作”进行了出色的概述。

## 获取源码
执行主仓库下的 `get_source.sh 命令，获取 OpenJDK Mercurial 仓库的全部内容。

    hg clone http://hg.openjdk.java.net/jdk8/jdk8 YourOpenJDK
    cd YourOpenJDK
    bash ./get_source.sh

当你拥有了所有的仓库之后，请记住，每个的仓库都是独立的。你也可以重新执行`./get_source.sh`，随时拉取所有仓库的最新变更集。这些内嵌的仓库被称作“forest”（树形结构），有多种方法可以为每个仓库执行相同的`hg`命令。例如，`make/scripts/hgforest.sh`脚本就可以为每个仓库执行相同的`hg`命令，如下：

    cd YourOpenJDK
    bash ./make/scripts/hgforest.sh status

## 仓库结构
仓库及其内容概述

| 仓库      | 内容概述                                                |
| --------- | ------------------------------------------------------- |
| .(root)   | 通用配置与 makefile 逻辑                                |
| hotspot   | 构建 OpenJDK Hotspot 虚拟机所需的源代码和 make 文件     |
| langtools | OpenJDK javac 和 language tools 的源代码                |
| jdk       | 构建OpenJDK运行时库与其它文件所需的的源代码和 make 文件 |
| jaxp      | OpenJDK JAXP 功能的源代码                               |
| jaxws     | OpenJDK JAX-WS 功能的源代码                             |
| corba     | OpenJDK Corba 功能的源代码                              |
| nashorn   | OpenJDK JavaScript 功能的源代码                         |

## 源码规范
一些基本的规范：
- 源文件中空格的使用是受限制的：不能有制表符、行末不能有空格、文件不能以一个以上的空行结束（源文件包括.java， .c， .h， .cpp， 和 .hpp 文件）
- 不应该将具有可执行权限的文件添加到源仓库中去
- 所有生成的文件需要与源代码控制系统所维护管理的文件保持隔离，生成的文件应当存放在顶层的 `build/` 目录下
- 默认的构建过程应该构建 product ，而不是其它。可供构建的选项如下：product（优化版本）、debug（未优化版本，附带-g的断言逻辑）、fastdebug（优化版本，附带-g的断言逻辑）
- 每个仓库必须存在`.hgignore` 文件，它必须包含 `build/`、`dist/`，可以包含`nbproject/private`等类似的文件夹，不能包含仓库中的 `src/` 、 `test/` 及其它在仓库管理范围内的任何内容
- 目录名和文件名不应该包含空格或非打印字符
- 生成的文件或二进制文件不应该添加到仓库中（包括 `javah` 输出）。对于这条规则有一些特例，特别是一些生成的 configure 脚本
- 不应该将不需要用于构建或测试的文件添加到仓库

# 构建
构建OpenJDK的第一步是确保系统本身已经拥有了构建OpenJDK所需要的一切。一旦系统设置完成，通常就不再需要执行这一步了。

构建OpenJDK现在可以通过运行一个 configure 脚本来完成，这个脚本会尝试查找并验证你是否已经拥有所需的一切，然后运行 make 命令，如下所示：

    bash ./configure
    make all

在可能的情况下， `configure` 文件会尝试在默认位置或组件指定的变量设置中来定位不同的组件，当正常的缺省设置失效或无法找到组件时，可能需要额外的 `configure` 选项来帮助 `configure` 找到构建所需的工具，如果构建过程中提示缺失软件包，你可能需要在安装指定软件包之后重新执行构建操作。

注意：`configure` 脚本文件没有执行权限，需要使用 `bash` 命令显式地运行它，请参看源码规范

## 系统设置
在尝试使用操作系统去构建 OpenJDK 之前，需要进行一些非常基本的系统设置。对所有操作系统来说：
- 请确保 GNU make 工具的版本大于或等于3.8.1，执行"`make -version`"来查看make的版本
- 安装一个引导 JDK。所有的 OpenJDK 构建都需要访问一个先前发布的 JDK ，这个 JDK 叫作 `bootstrap JDK` 或者 `boot JDK` 。一般的规则是，这个引导 JDK 必须是 JDK 上一个主要版本的实例。此外，可能需要使用一个特定的 update 级别或更高级别的引导 JDK来完成构建操作。
  
    构建 JDK8 需要使用 Update7 或以上版本的 JDK7， JDK8 的开发者不应该把 JDK8 当作引导 JDK ， 以确保由 JDK7 所构建的系统部分不会引入 JDK8 的依赖

    JDK7 二进制文件可以在 Oracle的 [JDK 7 下载站点](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 下载。构建过程中，引导 JDK 能够正常访问是非常重要的，你应该将引导 JDK 的 `bin` 路径添加到 `PATH` 环境变量中去。如果 `configure` 找不到这个引导 JDK ，你可能需要使用 `configure` 选项 `--with-boot-jdk` 来指定它的位置
- 确保 GNU make、引导 JDK 和编译器都位于你的 PATH 环境变量中

针对特定系统：
（仅翻译了Linux，Sloaris、Windows、Mac OS X 都没有翻译）

### Linux

安装所有需要的软件开发包，包括alsa、freetype、cups和xrender

使用 Linux 时，尽量使用系统包，而不是自己构建或从其他地方获取。大多数Linux都可以使用系统包。

注意，有些 Linux 系统有预先设置环境变量的习惯，例如，在Linux 系统中安装完 JDK 后， `JAVA_HOME` 可能会预先定义到你的环境变量中。你可能需要 unset `JAVA_HOME`。建议运行 `env` 命令，并验证你从系统默认设置中获取的环境变量对于构建 OpenJDK 是否有意义。

## Configure
`configure` 脚本的基本调用方法如下：

    bash ./configure [options]

上述操作将会创建一个含有 "configuration" 的输出目录，并为构建结果设置一个子目录，构建结果目录一般看起来像：

    build/linux-x64-normal-server-release

`configure` 会计算出你正在哪个系统上运行，以及全部所需构建组件位于何处。如果你已经安装了构建所需的所有先决条件，它就会找到所有内容。如果不能自动检测到任何组件，它就会退出并告诉你问题所在。发生这种情况时，请阅读下面的配置选项中的更多内容。

一些例子：

| 描述                       | Configure 命令                                               |
| -------------------------- | ------------------------------------------------------------ |
| 含有指定freetype的32位版本 | `bash ./configure --with-freetype=/cygdrive/c/freetype-i586 --with-target-bits=32` |
| fastdebug级别的64位版本    | `bash ./configure --enable-debug --with-target-bits=64`      |

### 配置选项

OpenJDK `configure` 选项的完整细节可以通过以下命令获取：

    bash ./configure --help=short

使用 `-help` 能看到所有可用的 `configure` 选项，你可以生成任意数量的不同配置，如debug、release、32、64等等。一些比较常用的 `configure` 选项如下：

| Configure 选项                 | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| --enable-debug                 | 设置debug级别为fastdebug （这是`--with-debug-level=fastdebug`的简写）。 |
| --with-alsa=path               | 设置ALSA（Linux声音驱动程序）的位置。<br>在Linux中，需要使用大于或等于0.9.1版本的 ALSA 文件来构建OpenJDK.。这些 Linux 文件通常可以从“libasound”开发包的“alsa”中获得，强烈建议使用由当前Linux发行版所提供的软件包。 |
| --with-boot-jdk=path           | 选择引导 JDK。                                               |
| --with-boot-jdk-jvmargs="args" | 提供用于运行引导JDK的JVM选项。                               |
| --with-cacerts=path            | 选择cacerts文件的路径。<br> 参看 [证书颁发机构](https://zh.wikipedia.org/wiki/%E8%AF%81%E4%B9%A6%E9%A2%81%E5%8F%91%E6%9C%BA%E6%9E%84) 以更好地理解证书授权概念。<br>“cacerts”文件表示CA证书的全局密钥库，在JDK和JRE二进制包中，“cacerts”文件包含来自几个公共CA的根CA证书(公共CA包括VeriSign、Thawte和Baltimore等)。<br>源文件包含了一个没有CA根证书的cacerts文件。正式构建时需要获得每个公共CA的许可，并将证书包含到它们自己的自定义cacerts文件中。<br>如果cacerts文件没有正确包含公共CA的许可，将导致运行时证书链的验证异常。默认情况下，会提供一个空的cacerts文件，这对大多数JDK开发人员来说应该没有问题。 |
| --with-cups=path               | 选择CUPS安装路径<br>在Solaris和Linux上构建OpenJDK需要CUPS(UNIX通用打印系统)头文件。Solaris中，可以通过安装Solaris Software Companion的CD/DVD中的SFWcups软件包来获得头文件，这些头文件通常会安装到目录/opt/sfw/cups中。<br>CUPS头文件可以从[www.cups.org](www.cups.org)下载。 |
| --with-cups-include=path       | 选择CUPS头文件路径。                                         |
| --with-debug-level=level       | 选择调试信息信息级别，包括release、fastdebug及slowdebug。    |
| --with-dev-kit=path            | 选择编译与开发工具的路径。                                   |
| --with-freetype=path           | 选择要使用的freetype文件。<br>预期在`lib/`目录下有freetype的库文件，在`include/`目录下有freetype的头文件。<br> freetype的版本需要大于或等于2.3。在Unix系统上，所需的文件可能已经内置了，但你仍然可能需要升级它们。注意，freetype必须是同时包含库文件和头文件的开发版本。<br>你可以从[FreeType 站点](http://www.freetype.org/)下载最新的FreeType<br>从头开始构建freetype 2库也是可以的，但是在Windows上构建可能需要参考[Windows freetype DLL构建说明](http://freetype.freedesktop.org/wiki/FreeType_DLL)<br>注意，在默认情况下，由于许可限制原因，FreeType构建时禁用了字节码提示支持。在这种情况下，文字的外观与大小将与Sun的JDK官方版本有所差异。有关更多信息，请参见[SourceForge FreeType2主页](http://freetype.sourceforge.net/freetype2/index.html)。 |
| --with-import-hotspot=path     | 选择以前版本的hotspot二进制文件的位置，以避免构建hotspot。   |
| --with-target-bits=arg         | 选择构建位数，包括32与64。                                   |
| --with-jvm-variants=variants   | 选择要构建的JVM变体，包括server, client, kernel, zero 和 zeroshark，多个选项用逗号分割。 |
| --with-memory-size=size        | 选择 GNU make 可运用的 RAM 上限。                            |
| --with-msvcr-dll=path          | 选择msvcr100.dll文件的位置， 这个文件是 Visual Studio的C/C++运行时库，在使用Windows构建时会用到。<br>这个文件通常是从Visual Studio 2010的redist目录中自动获得的。 |
| --with-num-cores=cores         | 选择要使用的内核数量（处理器数量或CPU数量）。                |
| --with-x=path                  | 选择X11和xrender文件的路径。<br>在Solaris和Linux上构建OpenJDK时需要使用XRender的扩展头文件，Linux头文件通常可以从"Xrender"开发包中获得，建议使用由当前Linux发行版所提供的软件包。<br>Solaris的XRender头文件和和其它X11头文件在新版本的Solaris的SFWxwinc包中，它们位于`/usr/X11/include/X11/extensions/Xrender.h`或`/usr/openwin/share/include/X11/extensions/Xrender.h`。 |

## Make

`make` 的基本调用方法如下：

    make all

执行这个命令将会在构建结果目录下开始构建，构建结果目录由`configure`脚本生成

运行`make help`可以获取更多支持的target信息

以下是一些大家普遍感兴趣的target:

| target     | 描述                                                        |
| ---------- | ----------------------------------------------------------- |
| 空         | 构建所有内容，但不包括镜像                                  |
| all        | 构建所有内容，且包括镜像                                    |
| all-conf   | 构建所有配置                                                |
| images     | 构建完整的j2sdk和j2re镜像                                   |
| install    | 将生成的镜像安装到本地，安装路径一般是`/usr/local`          |
| clean      | 删除所有由`make`生成的文件，不包括由`configure`生成的文件   |
| dist-clean | 删除所有由`make`和`configure`生成的文件（基本上是重置配置） |
| help       | 提供一些make命令的帮助，包含了一些常用的target              |

# 测试
构建完成之后，你应该可以在输出目录的`j2sdk-image`子目录下看到生成的二进制文件和其它相关联的文件。特别是，`build/*/images/j2sdk-image/bin`目录应该包含了当前配置的OpenJDK工具的可执行文件。

如果需要测试工具`jtreg`的话，你可以参见[jtreg站点](http://openjdk.java.net/jtreg/)。仓库中提供的回归测试可以使用以下命令运行：

    cd test && make PRODUCT_HOME=`pwd`/../build/*/images/j2sdk-image all

# 附录A：提示和技巧

## 常见问题解答

## 构建性能技巧

## 故障排查

# 附录B：GNU-Make信息

# 附录C：构建环境
