---
title: for_newer
description: 
categories:
tags: 
grammar_cjkRuby: true
---


## 今晚要讲的内容
冯诺依曼体系结构
内存-栈
CPU-寄存器
函数调用-栈帧与调用约定
系统调用
常见汇编指令

## 目标
+  用汇编写个 hello-world

----

## 冯诺依曼体系结构

![冯-诺依曼](https://www.github.com/Byzero512/blog_img/raw/master/1539078974475.png)

----

## 运行一个程序的背后 I

![load](https://www.github.com/Byzero512/blog_img/raw/master/1539089345619.png)

----

## 运行一个程序的背后 II

![load2](https://www.github.com/Byzero512/blog_img/raw/master/1539089526459.png)

----

## 主存

![main_memory](https://www.github.com/Byzero512/blog_img/raw/master/1539079587238.png)

----

## 栈
+ 栈其实就是一段连续的内存空间
+ 从高地址往低地址延伸

![stack](https://www.github.com/Byzero512/blog_img/raw/master/1539080073382.png)

- psuh pop

----

## 寄存器


![register](https://www.github.com/Byzero512/blog_img/raw/master/1.png "1")

----

## 今晚会讲到的一些寄存器
- rip: PC
- rsp: 栈顶指针寄存器
- rbp: 基址寄存器
- rflag: 标志寄存器
- GPR: rax,rdi,rsi,rdx,rcx,r8,r9...
- ....

----


 ### demo1
 
 + 看寄存器

----

## 汇编指令 I -- push,pop 
+ 入栈: push
+ 出栈: pop

----

### 入栈

![push](https://www.github.com/Byzero512/blog_img/raw/master/1539085143415.png)

----

### 出栈

![pop](https://www.github.com/Byzero512/blog_img/raw/master/1539085365868.png)

----


#### 汇编前置-声明变量
```cpp
#include<iostream>
using namespace std;
char char_data=1;
int int_data=1;
int int_array_data[3]={1,1,1};
char char_bss;
int int_bss;
int int_array_bss[3];

int main(){}
```
```x86asm
section .data
char_data db 1
int_data dd 1
int_array_data dd 1,1,1

section .bss
char_bss resb 1
int_bss resw 1
int_array resw 3
```
----

#### 变量大小
byte
word
dword
qword

----

## 数据移动指令: mov
+ 数据存在寄存器与内存中
+ 两个操作数大小应该一样

![data-where](https://www.github.com/Byzero512/blog_img/raw/master/1539149605842.png)

----

### demo2
> 将数据在CPU,内存中移动

----

## 函数调用
+ jmp, call
+ 函数调用约定
+ 栈帧

----

###  demo3-jmp, call

call=push rip+jmp

----

### 函数调用约定
```c++
int main()
{
	char buf[]="Hello World\n";
	write(1,buf,12);
	return 0;
}

```

----

### 函数调用约定
+ 64位的程序, 函数调用约定为 fastcall
+ fastcall中约定: 函数的前6个参数从左到右放在寄存器 rdi, rsi, rdx,rcx,r8,r9. 超出六个的参数, 从右到左 push到栈里面, 函数的返回值放在rax中
	- 例如调用 write(1,buf,11)
	- 1 放在了 rdi, buf放在了rsi, 11放在rdx中

+ 对于32位的程序
+ 待会看demo4

----

### 栈帧
+ call func
+ push rbp
+ mov rbp,rsp
+ 栈帧开始

----

#### call func
![call](https://www.github.com/Byzero512/blog_img/raw/master/1539150796654.png)

----

#### push rbp

![push_rbp](https://www.github.com/Byzero512/blog_img/raw/master/1539151095032.png)

----

#### mov rbp,rsp

![rbp_rsp](https://www.github.com/Byzero512/blog_img/raw/master/1539151397024.png)

----


#### demo4
1. 看一下函数调用时,参数放在哪里
2. 看下函数调用的过程与栈的变化

----

### 系统调用syscall
+ 操作系统提供给进程的一系列API
+ 系统调用类似于函数调用, 只是函数调用时调用函数来完成任务, 而系统调用则是请求操作系统来完成任务

![userandkernel](https://www.github.com/Byzero512/blog_img/raw/master/1539149165919.png)

----

### 汇编怎么使用syscall
1. syscall table
2. set rax
3. set other Register
4. syscall

----

### demo5- 用汇编写hello world
+ 先回忆以下之前学过的指令
	+ mov, add,sub,push,pop,jmp,call
+ 设置寄存器: mov
+ 设置好寄存器之后: syscall

----

### 其他汇编指令

----

#### lea
+ 获得var_name这个变量的地址

```x86asm
lea rax,[var_name]
```
----

### 其他需要了解的知识 I
1. flag寄存器
2. cmp,test 指令
3. 有条件的jmp指令

+ 用以上内容实现 高级编程语言的 if 语句和 循环语句

----

















