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

----

 ### demo1

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

----

## 数据移动指令: mov
> 数据存在寄存器与内存中
> 两个操作数大小应该一样

![mov](https://www.github.com/Byzero512/blog_img/raw/master/1539085603626.png)

----

### demo2

----

## 函数调用
+ jmp, call
+ 函数调用约定
+ 栈帧

----

### jmp, call

call=push rip+jmp

+ demo3

----

### 函数调用约定
```c++
int main()
{
	char buf[]="hello_world";
	write(1,buf,11);
	return 0;
}

```








