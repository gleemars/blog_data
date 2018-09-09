---
title: readme.md
description: 
categories:
tags: 
grammar_cjkRuby: true
---

### 脚本安装方法
> 下载 brk_wch.py 到本地
> 执行 echo "source ------/brk_wch.py" > ~/.gdbinit
> "-----"是 brk_wch.py 的所在目录,执行pwd可看到


### 为什么需要这个脚本
> 在我们调试开了pie保护的程序时,ida只能看到偏移,如果我们想下断点或者看程序bss段里面的变量时,可能需要先使用 vmmap 得到程序加载地址,再加上偏移才能下断点或者查看某个内存里面的变量
> 而使用这个脚本,会给gdb添加四条命令:
- b64   offset   在64位程序中下断点
- b32   offset   在32位程序中下断点
- x64   offset   相当于 x/20gx addr
- x32   offset   相当于 x/20wx addr
- offset是在ida里面看到的数字,千万不要加 0x,作者已经经过转换,如看到的数字为13,offset就是13,不要是0x13

### 脚本使用方法

