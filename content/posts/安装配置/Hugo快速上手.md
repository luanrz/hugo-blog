---
title: "Hugo快速上手"
date: 2022-10-17T20:50:15+08:00
---

Hugo是使用Go语言实现的静态博客生成工具，它的构建速度比Hexo更快。

## 一、安装Hugo

以Linux与Mac为例，Hugo安装指令如下所示：

```shell
pacman -S hugo # ArchLinux
```
```shell
brew install hugo #MacOS
```

在命令行输入`hugo version`，正确显示了版本号，即证明安装成功了。更详细的安装过程可参见[hugo官方安装教程](https://gohugo.io/getting-started/installing/)

## 二、初始Hugo化项目

### （一）新建站点

```shell
hugo new site hugodemo
cd hugodemo 
git init
```

### （二）新建文章

```shell
hugo new posts/hello.md
```

新建文章时，会依据`archetypes/default.md`为模版创建文件，文件将被放在`content`文件夹下面，`hello.md`的内容如下所示：

```
---
title: "Hello"
date: 2022-10-17T21:07:47+08:00
draft: true
---
```

### （三）增加主题

主题位于`themes`文件夹下，可以使用Git子模块导入喜欢的主题，以`PaperMod`主题为例：

```shell
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git submodule update --init --recursive # 仅在后续重新拉取hugodemo项目时，才需要同步跟新此git模块
```

### （四）启动Hugo

```shell
hugo server -t PaperMod -D
```
server指令标识启动Hugo服务。

`-t`(`--theme`)指定了主题名，后续也可以在配置文件中指定主题名为`PaperMod`。

`-D`(`--buildDrafts`)表示编译文章头部的`draft`标识为`true`的草稿文章，刚刚创建的hello.md就是一篇草稿文章。

### （五）访问站点

在终端执行：`curl http://localhost:1313`，或在浏览器中访问`http://localhost:1313`，即可看到应用启动成功，刚刚写的第一篇hello文章显示在了首页。

## 三、配置PaperMod主题

未经配置的PaperMod还不太完整，下面将简单介绍PaperMod主题的配置过程。

首先，将根目录的`config.toml`重命名为`config.yml`，并将其中的`=` 换成`:`。随后，继续编辑`config.yml`，完整的配置文件如下：

```yml
baseURL: "https://blog.luanrz.cn/"
languageCode: "zh-cn"
title: luanrz's blog
paginate: 5
theme: PaperMod

# 右上角菜单项
menu:
  main:
    - identifier: home
      name: 首页
      url: /
      weight: 10
    - identifier: search
      name: 搜索
      url: /search/ 
      weight: 20      
    - identifier: archives
      name: 归档
      url: /archives/
      weight: 30
    - identifier: categories
      name: 分类
      url: /categories/
      weight: 40      
    - identifier: tags
      name: 标签
      url: /tags/
      weight: 50

# 与Search搭配使用
outputs:
    home:
        - HTML
        - RSS
        - JSON # 必填，不加此节点Search无法使用

params:
  defaultTheme: auto # 主题颜色：auto、dark、light
  disableSpecial1stPost: true # 在Regular模式下禁用Home-Info区域，该开关在Home-Info模式和Profile模式无意义
  hideSummary: true # 隐藏文章列表项的概述信息
  hidemeta: true # 隐藏文章列表项的元数据，如：ReadingTime、WordCount等等
  ShowReadingTime: true # 显示读完的预计时间（hidemeta为false才有意义）
  ShowWordCount: true # 显示统计字数（hidemeta为false才有意义）
  ShowCodeCopyButtons: true # 显示代码中的复制按钮
  ShowPostNavLinks: true # 显示文章中上一篇和下一篇的链接
  ShowBreadCrumbs: false # 显示面包屑导航
  showtoc: false # 显示目录
  tocopen: false # 显示的目录默认处于打开状态（showtoc为true才有意义）
  label:
    text: "luanrz's blog" # 左上角logo区域的文字，如果不填将以全局的title值为准

  # Home-Info
  homeInfoParams:
    Title: "luanrz的个人博客" # Home-Info区域的标题
    Content: "“心有所向，日服一日，必有精进”" # Home-Info区域的内容
  socialIcons:
    - name: bilibili # B站
      url: "https://space.bilibili.com/31949997"
    - name: github # Github
      url: "https://github.com/luanrz"

```

上述配置中有两个菜单项需要新建对应的文件，如下所示：

新建archives.md以支持归档功能

```shell
vim content/archive.md
```

```
---
title: "Archives"
layout: "archives"
url: "/archives/"
---
```

新建search.md以支持搜索功能

```shell
vim content/search.md
```

```
---
title: "Search"
layout: "search"
placeholder: "输入关键字搜索文章"
---
```

最后，再次重启项目，可以看到看到首页已经焕然一新。

## 四、Hexo的迁移建议

此章节适用于希望将老的Hexo的博客全部迁移到新的Hugo的场景。

### （一）文章结构微调

首先，Hexo文章头部的描述信息大多数可以直接平移，无需额外修改。但Hugo的description不支持markdown及html语法，建议将对应的description信息挪到文章正文。

同时，Hugo不支持Hexo的Tabs标签语法，可以删掉对应的Tabs标签或使用hugo对应Shortcodes语法。

### （二）文章内部超链接

Hugo中Markdown对应的标题生成超链接锚点时，会执行下述操作：将大写字母替换成小写字母、将`、`与`（）`等符号去掉。

以`## （一）PaperMod`为例，它的超链接锚点会变成`一papermod`，如果要在文章内部引用它，可以使用`[paper](一papermod)`

### （三）支持PlantUML

PlantUML是一种文本画图的工具，Hexo的Next主题原生支持PlantUML。Hugo现在还没有原生支持PlantUML，根据其github的讨论记录，可能后续会支持。

现在的解决方案是自定义js，将PlantUML文本传输到`http://plantuml.com/`在线生成图片并回写到页面。篇幅受限，具体实现方法就不展示了。


> 参考文档

1. [Hugo官方入门手册](https://gohugo.io/getting-started/quick-start/)
2. [PaperMod安装教程](https://adityatelange.github.io/hugo-PaperMod/posts/papermod/papermod-installation/)