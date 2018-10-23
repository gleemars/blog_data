---
title: for_newer_10_25
description: 
categories:
tags: 
grammar_cjkRuby: true
---

## 今晚没有flag，只有helloworld

----

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


## 其他
1. 操作系统： 管理软件与硬件资源
2. 程序：代码
3. 进程： 运行的程序

![softandhard](https://www.github.com/Byzero512/blog_img/raw/master/1540269658137.png)

----

## 运行一个程序的背后 I

![load](https://www.github.com/Byzero512/blog_img/raw/master/1539089345619.png)

----

## 运行一个程序的背后 II

![load2](https://www.github.com/Byzero512/blog_img/raw/master/1539089526459.png)

----

## 硬盘、外存

----

## 主存

![main_memory](https://www.github.com/Byzero512/blog_img/raw/master/1539079587238.png)

----

## 虚拟内存中的进程

![virtual_memory](https://www.github.com/Byzero512/blog_img/raw/master/1540270085406.png)

----

## 栈
+ 栈其实就是一段连续的内存空间
+ 从高地址往低地址延伸

![stack](https://www.github.com/Byzero512/blog_img/raw/master/1539080073382.png)

- psuh pop

----

### 入栈

![push](https://www.github.com/Byzero512/blog_img/raw/master/1539085143415.png)

----

### 出栈

![pop](https://www.github.com/Byzero512/blog_img/raw/master/1539085365868.png)

----

## 汇编指令 I -- push,pop 
+ 入栈: push
+ 出栈: pop

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
 
 + 寄存器： rip ....
 + 栈： rbp， rsp

----

## 正式开始讲汇编

----

## 程序与文件结构
1. 文件后缀： .exe...每一种后缀都对应着一种文件格式
2. 代码： 指令和数据
3. 数据： 初始化和未初始化

elf文件格式：
1. 文件头
2. .text
3. .bss ：没有初始化的全局变量
4. .data：已经初始化了的全局变量 

+ readelf 看一下
+ ida观察: demo0 

----

#### 汇编前置-声明变量I

+ 变量大小

byte=8bit
word=2byte
dword=2word
qword=2dword

+ 声明格式
1. 初始化的变量： varName varType varValue
		例如： Str db "abc"==char str[3]="abc";
2. 未初始化的变量： varName varType Count
	    例如：str resb 10==char str[10];

----

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
---

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

## 数据移动指令或者说赋值指令: mov
+ 数据存在寄存器与内存中
+ 两个操作数大小应该一样

![data-where](https://www.github.com/Byzero512/blog_img/raw/master/1539149605842.png)

----

### demo2
1. 将数据在CPU,内存中移动


----

## 汇编前置-函数声明
```cpp
int diy();
int main()
{
	diy();
	return 0;
}

```
---
```x86asm

section .text
global _start
_start:
	;function body
```

----

## task1： 用汇编实现一下的c语言程序

```cpp
char a;
char b[]='a';
int main()
{
	a=b;
}

```

----

## demo-answer1

----

## 函数调用 I

```cpp
int diy();
int main()
{
	diy();
	return 0;
}
int diy()
{
	cout<<1;
}
```
---

```x86asm
section .text
global _start
_start:
	call _diy
	
global _diy
_diy:
	;function bogy
```

----

## 函数调用 II
+ jmp, call
+ 函数调用约定
+ 栈帧

----

###  demo3

jmp
call=push rip+jmp

----

### 函数调用约定I
```c++
int main()
{
	char buf[]="Hello World\n";
	write(1,buf,12);
	return 0;
}

```

----

### 函数调用约定II
+ 64位的程序, 函数调用约定为 fastcall
+ fastcall中约定: 函数的前6个参数从左到右放在寄存器 rdi, rsi, rdx,rcx,r8,r9. 超出六个的参数, 从右到左 push到栈里面, 函数的返回值放在rax中
	- 例如调用 write(1,buf,11)
	- 1 放在了 rdi, buf放在了rsi, 11放在rdx中

---
+ 对于32位的程序, 参数放在栈里面
+ 例子待会看demo4

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

#### 返回
1. leave==mov rsp, rbp; pop rbp
2. ret ==pop rip

----

#### leave
![leave](https://www.github.com/Byzero512/blog_img/raw/master/1540276224508.png)

----

#### ret

![ret](https://www.github.com/Byzero512/blog_img/raw/master/1540276352171.png)

----

#### 对比一下对用函数前和调用函数后栈的变化

![beforeandafter](https://www.github.com/Byzero512/blog_img/raw/master/1540276485331.png)

----

#### 所以一个完整的函数框架应该是
```x86asm
section .text
global _start
_start:
	push rbp
	mov rbp,rsp
	call func; call=push rip+jmp func
	;--------------
	leave
	ret
	
global func
func:
	push rbp
	mov rbp,esp
	leave 
	ret

```

----

#### demo4
1. 看一下函数调用时,参数放在哪里
2. 看下函数调用以及返回
3. 栈的变化

----

#### 请大家完善一下 task1
1. 要求有栈的变化

----

## 到这里我们已经知道
1. 变量声明
2. 函数声明
3. 懂得如何赋值

---

问题来了：
1. 怎样子才能向屏幕输出“helloworld”呢？
	- 在c语言我们可以调用write或者printf函数
	- 在c++我们可以使用cout
	- 但是在汇编语言中没有这些函数，关键字
？？？？？？？？？？？？？？？？？？？？？？

----

## 系统调用syscall
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

### 用汇编写hello world
+ 先回忆以下之前学过的指令
	+ mov, add,sub,push,pop,jmp,call
+ 设置寄存器: mov
+ 设置好寄存器之后: syscall

----

### demo5
1. 汇编中的helloworld

----


### 其他汇编指令

----



#### lea
+ 获得var_name这个变量的地址

```x86asm
lea rax,[var_name]
```

----



## 今晚到这里，我们好像已经差不多会写简单地汇编了，但是。。。

----

## if语句与循环语句
1. cmp指令
2. test指令
3. rflag 寄存器
4. 有条件的跳转指令
5. loop指令

----

## if语句和循环语句
1. 他们的特点就是比较某个表达式之后再决定执行代码
2. 实现比较的指令是 cmp 和 test
3. 比较之后的结果（例如if（a==1））储存在 rflag 寄存器中的标志位中
4. 有条件跳转指令根据rflag的标志位，决定跳转的目的地址

----

### 这里简单地讲一下
1. 如果等于则执行……，如果不等于则执行……

----

### demo7
1. cmp op1，op2 
2. result=op1-op2
3. 根据结果试着rflag标志位
4. 例如 result==0，即两个操作数相同，则会设置rflag寄存器中的某一位（ZF）为1，否则ZF为0
5. 有条件跳转指令（JE）根据ZF是1还是0就会执行不同的代码，从而实现一个简单地if语句


















