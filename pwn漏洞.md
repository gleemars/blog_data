---
title: pwn漏洞
tags: 
grammar_cjkRuby: true
---

##  溢出
1. 数组溢出
2. 整数溢出

## 函数
1. 函数不能正确实现功能时, 程序不一定会abort
+ scanf('%d',&int_data): 如果输入的是 '+' 号等特殊符号, 函数既不会写入内容到 int_data, 也不会抛出异常

2. 函数特殊的地方
	+ strncat: 会在字符串后加'\x00', 可能会导致 null off by one
	+ open: 打开一个文件1023次之后, 就不能再打开了

## 残留: 如果觉得程序没问题, 很大可能是残留导致的漏洞.
1. 变量声明时未初始化 (特别是使用某些字符串函数的时候)


## 关于32位程序
> 在64位ubuntu下用gcc编译出32位的程序, main函数的栈变化很奇怪

```
lea ecx,[esp+4]       ; ecx=esp+4
and esp, 0FFFFFFF0h     ; esp 对齐
push dword ptr [ecx-4]   ; 
push ebp
mov ebp, esp
push ecx

sub esp,n      ; 创建栈帧

mov ecx,[ebp-4]
leave
lea esp, [ecx-4]
retn

```
```
-------------------------------
		ecx1
------------------------------- cur_bp
	      old_bp
------------------------------- 
		ip
------------------------------- sp2
	sp 0x10对齐区域
------------------------------- sp1
		ip
------------------------------- ecx1
```

> 这样子的栈帧变化, 为栈迁移提供了条件