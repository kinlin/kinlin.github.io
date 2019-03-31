---
layout:     post
title:      使用vim+cscope如何动态更新代码索引
subtitle:   
date:       2019-04-01
author:     LinJin
header-img: img/theworld.jpg
catalog: true
tags:                            
    - 技能
---

###前言

对于使用source insight的人而言，并不存在这个问题，因为source insight在文件变动时自动就会处理索引的更新。但是对于使用vim+cscope的同学而言，就没那么方便了。

试想一下，你正在修改代码，然后cscope找函数，发现由于修改更新了，找到的位置偏移了好几百行。。。这时候，退出再重新进入编辑。效率就极大降低了，
违背了我们用工具提高看代码效率的初衷。


###如何修改
- 在`.vimrc`文件中添加下面函数。之后每次在阅读代码的同时就能直接按`F12`更新代码索引了
```shell
"reset cscope out file
map <F12> : call ReConnectCscope()<cr>
func! ReConnectCscope()
exec "cs kill 0"
exec "!./generate.sh"
exec "set csprg=~/cscope.sh"
exec "cs add cscope.out"
endfunc

```
    - 另附上面会用到的`generate.sh`, 之所以使用generate.sh, 是为了自行过滤一些文件。
```shell
#!/bin/bash
date;
echo "ctags done......"
find -L . -name  "*.h" -o -name "*.c" -o -name "*.cc" -o -name "*.cpp" -o -name "*.py" > cscope.files;
find -L . -name "*.pl" >>cscope.files

date;
cscope -RCbq -i cscope.files;
ctags -R -L cscope.files;
echo "cscope done......"
date;

```

###使用vimrc文件定义快捷键执行更酷炫的操作
- 作为一名工程师，经常会需要`ssh`远程连接到其他机器，这种情况下可能会有其他用户也在使用vim看代码，这时候如果我们轻易改变vim的配置，那么`.vimrc`很快就会被改得面目全非。所以问题来了： **能不能定义一套配置，只有自己使用，其他人仍然使用默认的配置呢？**

> 当然有！

- **我们完全可以定义一套配置，通过快捷键生效！**
```shell
示例如下：

"Do this First : 首先需要安装plug脚本
"curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
"    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim

"把自己常用的一些脚本可以安装好
call plug#begin('~/.vim/plugged')
Plug 'vim-scripts/mru.vim'
Plug 'ctrlpvim/ctrlp.vim'
Plug 'tomasr/molokai'
Plug 'octol/vim-cpp-enhanced-highlight'
Plug 'lokaltog/vim-powerline'
Plug 't9md/vim-quickhl'
call plug#end()

"Magic time.执行F12快捷键后，我们可以配置色彩方案，配置插件设置。
"一切都变得不一样，专属于你自己的一套配置。所有操作习惯都可以继续。多么美好！
"some personal config
map <F12> : call PersonalConfigs()<cr>
func! PersonalConfigs()
exec "colorscheme molokai"
exec "set laststatus=2"
exec "set fillchars+=stl:\\ ,stlnc:\\"
exec "set encoding=utf-8"
exec "let g:cpp_class_scope_highlight = 1"
exec "let g:cpp_member_variable_highlight = 1"
exec "let g:cpp_experimental_template_highlight = 1"
exec "let g:cpp_concepts_highlight = 1"
exec "let g:Powerline_symbols = 'compatible'"
"exec "nmap <Space>m <Plug>(quickhl-manual-this)"
"exec "xmap <Space>m <Plug>(quickhl-manual-this)"
"exec "nmap <Space>M <Plug>(quickhl-manual-reset)"
"exec "xmap <Space>M <Plug>(quickhl-manual-reset)"
exec "set incsearch"
endfunc
```


###如何使得cscope查找字符时不区分大小写

通常情况下，我们查找函数定义时，需要提供精确的名字，但是我们很难记住一个字符串同时还记住它第几个字符是大小写。

cscope本身就具有忽略大小写的选项，我们需要打开即可。

- 首先我们在home目录`~`下创建一个脚本`cscope.sh`,并赋予执行权限
```shell
#/!bin/sh

cscope -C "$@"
```

- 然后修改vimrc如下
```shell
" cscope settings
if has("cscope")
"   set csprg=/usr/bin/cscope
"主要修改就是这句，将调用/usr/bin下的cscope改为调用cscope.sh脚本
    set csprg=~/cscope.sh
    set csto=0
    set cst
    set nocsverb
    " add any database in current directory
    if filereadable("cscope.out")
        cs add cscope.out
        " else add database pointed to by environment
    elseif $CSCOPE_DB != ""
        cs add $CSCOPE_DB
    endif
    set csverb
endif

```

- 现在可以试着搜索一个字符串，完全不用顾及大小写的问题，舒服多了。
