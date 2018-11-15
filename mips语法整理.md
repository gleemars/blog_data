---
title: mips语法整理
tags: 
categorires: 
author: 
top: 
---

## 前置
1. mips指令集中, 有很多指令实际上是并不是真正懂的原始指令, 而是类似于宏的存在, 汇编器编译时, 会把这类指令转换成(多条)原始指令

## 寄存器
1. $v0 返回值
2. $sp 栈顶指针寄存器
3. $fp ($s8) 栈帧基址寄存器

4. $ra 函数返回地址

> $0 永远为0
> $a0 -- $a3 参数寄存器
> $t0 -- $t9   临时寄存器
> $s0 -- $s7 (进入子函数)可能需要保存的寄存器

> 还有其他寄存器不介绍

## 汇编语言

### 变量声明
> 和arm类似
> varName: varType varValue
> varType有: .ascii, .asciiz, .byte, .half, .word, .quad, .space ....
```mipsasm
	.text
.global __start   # 是__start不是_start
__start:
	# body
	
	.data
hello:
	.asciiz "hello world"

```



## 系统调用
+ 调用号: $v0
+ 参数: $a0 -- ??
+ 系统调用指令: syscall
+ 返回值放 $v0


## 函数

### 函数调用约定
1. 保存环境(如果需要的话)
> $t0 - $t9 由调用者保存

2. 设置参数
> $a0 -- $a3; 多余的放栈上

3. 跳转
> jal jalr
> bal

4. 进入子函数, 创建栈帧, 保存上下文
5. 返回值放 $v0

### 栈帧
> $fp, $sp 以及局部变量的位置和 x86 类似, 但是有多一个 gp (暂时不知道是什么)
> 不需要保存 $ra 到栈的时候不会保存(类似于arm)

## 指令
+ des must always be a register.
+ src1 must always be a register.
+ reg2 must always be a register.
+ src2 may be either a register or a 32-bit integer
+ addr must be an valid address. 

### 算术指令
![ins1](https://www.github.com/Byzero512/blog_img/raw/master/1542283194746.png)

### 比较指令
![ins2](https://www.github.com/Byzero512/blog_img/raw/master/1542283230833.png)

### 分支指令
![ins3](https://www.github.com/Byzero512/blog_img/raw/master/1542283267925.png)

### 调转指令
![ins4](https://www.github.com/Byzero512/blog_img/raw/master/1542283294500.png)

### 数据操作指令
![ins5](https://www.github.com/Byzero512/blog_img/raw/master/1542283337136.png)

#### 1. load
![load_ins](https://www.github.com/Byzero512/blog_img/raw/master/1542283399424.png)

#### 2. store
![store_ins](https://www.github.com/Byzero512/blog_img/raw/master/1542283489296.png)

#### 3. mov
![mov_ins](https://www.github.com/Byzero512/blog_img/raw/master/1542283522273.png)