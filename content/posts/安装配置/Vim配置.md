---
title: Vim配置
date: 2018-11-20 17:05:00
categories:
- 安装配置
- 应用软件
tags: 
- Linux
description: 使用Vundle管理Vim插件与实用插件配置
---

# Vim配置

## 一、安装插件管理器Vundle

> 参见[在 Linux 上使用 Vundle 管理 Vim 插件](https://linux.cn/article-9416-1.html) 及[VundleVim/Vundle.vim](https://github.com/VundleVim/Vundle.vim)

- 下载Vundle

	`git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim`

- 配置Vundle

	`vim ~/.vimrc`

	```
	set nocompatible              " be iMproved, required
	filetype off                  " required

	" set the runtime path to include Vundle and initialize
	set rtp+=~/.vim/bundle/Vundle.vim
	call vundle#begin()
	" alternatively, pass a path where Vundle should install plugins
	"call vundle#begin('~/some/path/here')

	" let Vundle manage Vundle, required
	Plugin 'VundleVim/Vundle.vim'

	" The following are examples of different formats supported.
	" Keep Plugin commands between vundle#begin/end.

	" plugin on GitHub repo
	" Plugin 'tpope/vim-fugitive'
	" plugin from http://vim-scripts.org/vim/scripts.html
	" Plugin 'L9'

	" Git plugin not hosted on GitHub
	" Plugin 'git://git.wincent.com/command-t.git'

	" git repos on your local machine (i.e. when working on your own plugin)
	" Plugin 'file:///home/gmarik/path/to/plugin'

	" The sparkup vim script is in a subdirectory of this repo called vim.
	" Pass the path to set the runtimepath properly.
	" Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
	" Install L9 and avoid a Naming conflict if you've already installed a
	" different version somewhere else.
	" Plugin 'ascenator/L9', {'name': 'newL9'}

	" All of your Plugins must be added before the following line
	call vundle#end()            " required
	filetype plugin indent on    " required
	" To ignore plugin indent changes, instead use:
	"filetype plugin on
	"
	" Brief help
	" :PluginList       - lists configured plugins
	" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
	" :PluginSearch foo - searches for foo; append `!` to refresh local cache
	" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
	"
	" see :h vundle for more details or wiki for FAQ
	" Put your non-Plugin stuff after this line
	```

## 二、实用插件配置
- 编辑插件地址：在~/.vimrc文件的`call vundle#begin()`与`call vundle#end() `之间写入`Plugin '......'	`
- 编辑插件配置：在~/.vimrc文件末尾写入配置信息
- 安装插件：vim --> : -->PluginInstall

- [语法高亮与行号显示](https://blog.csdn.net/chuanj1985/article/details/6873830)
	```   
	set nu!                                          "显示行号
	syntax on                                    "语法高亮度显示

	```

- [NERDTree](https://github.com/scrooloose/nerdtree "文件系统资源管理器")
	```
	Plugin 'scrooloose/nerdtree'	
	" --------------------NerdTree--------------------
	" 设置NerdTree,按F3即可显示或隐藏NerdTree区域
	map <F3> :NERDTreeMirror<CR>
	map <F3> :NERDTreeToggle<CR>
	"打开vim时自动打开NERDTree
	autocmd vimenter * NERDTree
	```

- [vim-markdown](https://github.com/plasticboy/vim-markdown "Markdown")
	```
	Plugin 'godlygeek/tabular'
	Plugin 'plasticboy/vim-markdown'
	```

- [vim-instant-markdown](https://github.com/suan/vim-instant-markdown "Markdown实时预览")
	```
	Plugin 'suan/vim-instant-markdown'
	```

- [vim-table-mode](https://github.com/dhruvasagar/vim-table-mode)
	```
	Plugin 'dhruvasagar/vim-table-mode'	"使用\tm开启Table Model
	```

- [auto-pairs](https://github.com/jiangmiao/auto-pairs "自动插入和格式化方括号和圆括号")
	```
	Plugin 'jiangmiao/auto-pairs'
	```


- [syntastic](https://github.com/vim-syntastic/syntastic "语法检测")
	```
	Plugin 'scrooloose/syntastic'
	" --------------------syntastic--------------------
	" syntastic官方推荐的默认设置
	set statusline+=%#warningmsg#
	set statusline+=%{SyntasticStatuslineFlag()}
	set statusline+=%*
	let g:syntastic_always_populate_loc_list = 1
	let g:syntastic_auto_loc_list = 1
	let g:syntastic_check_on_open = 1
	let g:syntastic_check_on_wq = 0
	```

- [youcompleteme](https://github.com/valloric/youcompleteme "代码补全")
	```
	Plugin 'valloric/youcompleteme'
	```

- [supertab](https://github.com/ervandew/supertab "代码补全")
	```
	Plugin 'ervandew/supertab'
	```

- [ NERD Commenter](https://github.com/scrooloose/nerdcommenter "注释")
	```
	Plugin 'scrooloose/nerdcommenter'
	```

