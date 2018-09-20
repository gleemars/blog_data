---
title: x86 和 x64 汇编与机器码中的 jmp 与 call
description: 
categories:
tags: 
grammar_cjkRuby: true
---

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
> 兼容前面的指令
### jmp
```x86asm
;通过偏移地址进行转移
jmp <offset_ime>; 机器码为: E9 <offset>; 长度为 5

;和上面的指令一样
jmp <offset> <imd>; 机器码为 E9 <offset>; 长度为 5

;通过绝对地址进行转移
jmp <addr>; 机器码为: FF 25 <addr>; 长度为 6

```
### call

```x86asm
;通过偏移地址进行转移
call <offset_ime>; 机器码为: E8 <offset>; 长度为 5

;和上面的指令一样
call <offset> <imd>; 机器码为 E8 <offset>; 长度为 5

;通过绝对地址进行转移
call [addr]; 机器码为: FF 15 <addr>
```


## amd64
> 兼容前面的指令

### jmp
```x86asm
;通过偏移地址进行转移
jmp <offset_imd>; 机器码为: E9 offset; 长度为 5

;和上面的指令一样
jmp <offset> <imd>; 机器码为 E9 <offset>; 长度为 5

;64位jmp语句由于指令长度限制,只能现将绝对地址放入寄存器中,再转移
;通过寄存器存放的绝对地址进行转移
jmp <reg>; 机器码为 FF <reg>; 长度为 2
```

### call
```x86asm
;通过偏移地址进行转移
call <offset_ime>; 机器码为: E8 <offset>; 长度为 5

;和上面的指令一样
call <offset> <imd>; 机器码为 E8 <offset>; 长度为 5

;64位jmp语句由于指令长度限制,只能现将绝对地址放入寄存器中,再转移
;通过寄存器存放的绝对地址进行转移
call <reg>; 机器码为: FF <addr>; 长度为 2

```
