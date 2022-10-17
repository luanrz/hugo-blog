---
title: IDEA插件开发环境搭建
date: 2020-05-03 12:18:00
categories:
- 安装配置
- 应用软件
tags: 
- IDEA
---

本文介绍了手动下载并安装IDEA插件开发环境依赖的过程, 以解决IDEA插件开发环境过程中下载Gradle包过慢的问题<br/>如果你的网络环境可以正常下载依赖包, 本文对你来说是通篇废话, 请忽略.

# IDEA插件开发环境搭建

## 一、依赖

由于在国下载内Gradle包的速度较慢，部分大文件会因为下载时间过长连接超时导致下载失败。

在Idea插件项目中，主要会下载以下三个大文件：

1. ideaIC-2020.1.zip	（500M左右）
2. ideaIC-2020.1-sources.jar	（100M左右）
3. jbr-11_0_6-linux-x64-b765.25.tar.gz	（100M左右）

下面是`ideaIC-2020.1.zip`依赖的解决过程，其它依赖的解决过程与之类似。

### 1. 获取下载链接并手动下载依赖包

在命令行下进入Gradle项目根目录，执行下述命令：

`./gradlew compileDebugSource --stacktrace -info`

在上述日志中找到下载链接，通过迅雷下载。（你也可以直接通过 [附录](#附各依赖包的路径) 部分下载）

### 2. 获取依赖包的SHA1码

使用`sha1sum`命令获取文件的SHA1码。

```shell
sha1sum ideaIC-2020.1.zip 
```

执行上述命令后，将得到：

```shell
cbeeb1f1aebd4c9ea8fb5ab990c5904a676fc41a  ideaIC-2020.1.zip
```

`cbeeb1f1aebd4c9ea8fb5ab990c5904a676fc41a`就是`ideaIC-2020.1.zip`的SHA1码。

### 3. 在gradle本地缓存的指定路径下创建SHA1码同名文件夹，将并将依赖包移动至此

- 进入gradle本地缓存的路径

  `cd ~/.gradle/caches/modules-2/files-2.1`

- 进入`ideaIC-2020.1.zip`的所在路径

  `cd com.jetbrains.intellij.idea/ideaIC/2020.1`

- 创建SHA1码同名文件夹

  `mkdir cbeeb1f1aebd4c9ea8fb5ab990c5904a676fc41a` 

- 移动依赖包

  `cp ~/Downloads/ideaIC-2020.1.zip cbeeb1f1aebd4c9ea8fb5ab990c5904a676fc41a`

### 附：各依赖包的路径

1. `com.jetbrains.intellij.idea/ideaIC/2020.1/xxx/ideaIC-2020.1.zip` [下载](https://cache-redirector.jetbrains.com/www.jetbrains.com/intellij-repository/releases/com/jetbrains/intellij/idea/ideaIC/2020.1/ideaIC-2020.1.zip)
2. `com.jetbrains.intellij.idea/ideaIC/2020.1/xxx/ideaIC-2020.1-sources.jar` [下载](https://cache-redirector.jetbrains.com/www.jetbrains.com/intellij-repository/releases/com/jetbrains/intellij/idea/ideaIC/2020.1/ideaIC-2020.1-sources.jar)
3. `com.jetbrains/jbre/jbr-11_0_6-linux-x64-b765.25.tar.gz` [下载](https://cache-redirector.jetbrains.com/jetbrains.bintray.com/intellij-jbr/jbr-11_0_6-linux-x64-b765.25.tar.gz)
