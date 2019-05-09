---
layout:     post
title:      vim打开大文件时无法高亮语法
subtitle:   
date:       2019-05-01
author:     kinlin
header-img: img/theworld.jpg
catalog: true
tags:                            
    - vim
---


### 问题

使用vim打开非常大的文件时，具有一定的优势，尤其是通过ssh远程连接客户代码，但是经常会碰到语法无法高亮的情形。

这时候，如果往上翻到高亮处，再逐页往下，就能高亮。这样看代码实在有点麻烦。在我的机器上，如果打开一个20000行左右的源码，

就有可能出现后半段无法高亮的情形。尤其是在跳转时，突然从高亮代码跳转到一片灰蒙蒙的代码，非常影响效率和心情。


### 解决

google后发现，社区上其实已经有了讨论。

https://github.com/vim/vim/issues/2790

因为对于vim的渲染而言，会有一个时间限制，如果超出限制当页未能完成，就停止了。这时候就有了第一个解法：

增加渲染时间到最大： `set redrawtime=10000`

遗憾的是，在我这行不通，可能是即便设置了最大时间，依然无法完成。这样做反而对打开文件性能有巨大的影响。明显变慢，因为渲染时间更长了。

那还有没有其他方法？ 有，就是强制刷新本页，这样做对当前文件是有效的，能解决。

办法就是在vim里执行`syntax sync fromstart`

这样是能解的。不过在这过程中，发现`syntax on`和`syntax off`比较情形下，明显开了语法高亮时特别慢，不够流畅。

综合下来的解决方案就是，如下配置：

```
" vim卡顿 https://github.com/wklken/k-vim/issues/233
" 主要是因为syntax on. 如果syntax off会好很多
set re=1
set ttyfast
set lazyredraw

"map syntax sync fromstart
nmap tu :syntax sync fromstart<cr>
```

这样在打开时速度快，不卡顿。遇到大文件无法渲染完成时，只需要使用快捷键`tu`即可迅速完成。 毕竟过万行的代码是少数的，为了极少数大文件的

语法高亮而牺牲速度是不值得的。
