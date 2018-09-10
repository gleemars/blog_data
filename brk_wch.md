---
title: brk_ech_py.md
description: 
categories:
tags: 
grammar_cjkRuby: true
---

## 写在最前面的话
> 这个小脚本只是为了在做 pwn题 调试pie程序的时候方便下断点和查看程序自身的内存而写

## install
> 1. 下载 brk_wch.py 到本地
> 2. 执行 echo "source ------/brk_wch.py" > ~/.gdbinit
> 3. "-----"是 brk_wch.py 的所在目录,执行pwd可看到

> ps1: 作者是在 ubuntu 16.04 下写的脚本,并未在其它环境下测试过.
> ps2: 这个脚本需要 python 的 gdb 模块,ubuntu安装方法为:
> + sudo apt-get install gdb python2.7-dbg

---

## why need the script
> 在我们调试开了pie保护的程序时, ida只能看到偏移,如果我们想下断点或者看程序bss段里面的变量时,可能需要先使用 vmmap(gdb 有peda插件的情况下) 得到程序加载地址 proc_base ,再加上偏移 offset 才能下断点或者查看某个内存里面的变量

> 而使用这个脚本,会给gdb添加下面的两条命令:
> 1. **nb offset** : 通过 offset 下断点
> 2. **nx offset** : 通过 offset 看程序内存(注意只是 pie 程序本身的数据)
>    - nx offset 和 x/20wx (proc_base+offset) 或者 x/20gx (proc_base+offset) 的功能是一样的,只是 nx 可以自动找到 proc_base

---

### 小细节
> nb 123  和 nb 0x123 作用是一样的,都是在距离程序加载的基地址偏移 0x123 出下断点
> 同理 nx 123 和 nx 0x123 作用也是一样的
---
---
 ## how to use

### 在偏移为 0x12CC 的位置下一个断点

![example_1](https://www.github.com/Byzero512/blog_img/raw/master/nw/1536585213295.png)

> **在gdb中输入 "nb 12CC" 或者 "nb 0x12CC" 即可**

![example_2](https://www.github.com/Byzero512/blog_img/raw/master/nw/1536585533909.png)

![result_1](https://www.github.com/Byzero512/blog_img/raw/master/nw/1536585561322.png)
---
### 查看偏移为 0x2020C0 的变量的内容

![example21](https://www.github.com/Byzero512/blog_img/raw/master/nw/1536585665254.png)

> ** 在 gdb 中输入 "nx 2020C0" 或者 "nx 0x2020C0" 即可**

![example_result_2](https://www.github.com/Byzero512/blog_img/raw/master/nw/1536585938107.png)

## 写在最后的话
> 脚本持续更新中