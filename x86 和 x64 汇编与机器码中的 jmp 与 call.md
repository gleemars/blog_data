---
title: x86 和 x64 汇编与机器码中的 jmp 与 call
description: 
categories:
tags: 
grammar_cjkRuby: true
---


### 跳转方式
1. rip偏移 ff 25 
2. 距离程序基址的偏移
3. 绝对寻址

## Pre
### 8086
> 8086 中由于地址总线的限制和段寄存器的存在,jmp和call语句有 3 种
	+ 段内近转移
	+ 段内转移
	+ 远转移(段间转移)

### i386
> i386 由于地址总线扩展到32根, 所以 jmp 和 call 语句常见 2 种
	+ 通过相对地址转移
	+ 通过绝对地址转移

### amd64
> amd64 由于地址总线拓展到了64根,所以 jmp 和 call 语句常见 2 中
	+ 通过相对地址转移
	+ 通过绝对地址转移


## 8086

### jmp

```x86asm
;短转移
jmp short label; 机器码为: EB 

;段内转移,修改段寄存器: IP
jmp near ptr label; 机器码为: E9 

;段间转移,修改段寄存器: CS 和 IP
jmp far ptr label; 机器码为: EA 

```

### call
```x86asm

```


## i386
### jmp
```x86asm

;通过偏移进行转移,有短有近

;短距离
;机器码为 EB offset; 长度为 2, 即有 1byte 的机器码为 offset

jmp label
jmp short label
jmp offset label/addr

;==============================

;近距离
;机器码为: E9 offset; 长度为 5, 即有 4bytes 的机器码为 offset

jmp label 
jmp offset label/addr

;================================

;通过指针的地址进行转移
;机器码为: FF 25 [label/addr]; 长度为 6 bytes, 有 4bytes 的为指针的地址

jmp dword ptr ds:[label/addr]; 这四种情况的意思是: jmp *[label/addr]
jmp dword ptr [label/addr]
jmp ds:[label/addr];
jmp [label/addr]; 
```
> 上面的jmp情况中, 使用偏移远转移还是通过指针的地址进行转移, 应该由程序员写汇编是决定

?????????????????????其实对于pie程序是如何定位的?????????

### call

```x86asm

;偏移调用
;机器码为 E8 offset, 机器码长度为 5

call label/addr
call offset label/addr

;=======================

; 通过指针的绝对地址调用
; 机器码为: FF 15 [label/addr], 机器码长度为 6

call dword ptr ds:[label/addr]; 这四种情况都是 call *[label/addr]
call dword ptr [label/addr]
call ds:[label/addr]
call [label/addr]
```


## amd64
> 兼容前面的指令

### jmp
```x86asm

;短距离
;机器码为 EB offset; 长度为 2, 即有 1byte 的机器码为 offset

jmp label
jmp short label
jmp offset label/addr

;=========================

;近距离
;机器码为: E9 offset; 长度为 5, 即有 4bytes 的机器码为 offset

jmp label 
jmp offset label/addr

;==========================

;64位jmp语句由于指令长度限制,只能先将绝对地址放入寄存器中,再转移
;通过寄存器存放的绝对地址进行转移
jmp <reg>; 机器码为 FF <reg>; 长度为 2

;===========================
;远转移是怎么回事?

```

### call
> 远调用和32位的不同,主要是指令机器码的长度限制
```x86asm

;偏移调用
;机器码为 E8 offset, 机器码长度为 5

call label/addr
call offset label/addr

;远调用
lea rbx,[label/addr]
jmp rbx

```
