---
title: linux x64 汇编
description: 
categories:
tags: 
grammar_cjkRuby: true
---

## Preface

### 汇编器
> 本文使用 ysam 汇编器
> yasm -g dwarf2 -f elf64 example.asm -l example.lst

> nasm -g -f elf64 suorce.asm -l list_file -o output_file

#### ysam的参数
> -g dwarf2 在最后的obj文件中保留debug信息
> -f elf64 编译出64位的obj文件
> example.asm 输入文件
> -l 创建一个 list file , list file中有一些方便调试的信息

### 链接器ld
> ld -g  -o example example.o 

### 前置内容
![enter description here](https://www.github.com/Byzero512/blog_img/raw/master/1.png "1")

![enter description here](https://www.github.com/Byzero512/blog_img/raw/master/4.png "4")

![enter description here](https://www.github.com/Byzero512/blog_img/raw/master/1537271359304.png)

## asm,link,load,debugger

### 汇编

> 汇编过程其实是在 Assembler Directives(汇编器准则) 指示下进行 Two-Pass 的过程
> 每一次 pass 的行为都和编译器的设计有关 

#### first pass
+ 创建符号表: 有变量名和标签以及它们各自的地址
+ 展开宏
+ 计算常数表达式
+ 给汇编语句分配地址

#### second pass
+ 生成最终机器码(需要使用符号表)
+ 创建 list file (if requested)
+ 创建 obj file

### 链接

#### 外部引用 external reference

![extern_func](https://www.github.com/Byzero512/blog_img/raw/master/1537274714759.png)

---

## 变量语法

### 全局变量声明

1. const 常量声明
> var_name  equ  var_value
> such as: SIZE equ 1000;

2. 有初始化的变量声明
> 形式: var_name  dataType  var_value

![init_var_type_1](https://www.github.com/Byzero512/blog_img/raw/master/1537247003007.png)

![init_var_type_2](https://www.github.com/Byzero512/blog_img/raw/master/1537247014703.png)

> such as:  
> bVar db 10 //这么声明是一个整数
> bChar db "1" //这么声明是一个字符
> bStr db "10" //这么声明是一个字符串

![define_init_var_example](https://www.github.com/Byzero512/blog_img/raw/master/1537246736243.png)

3. 没有初始化的变量声明
> 形式: var_name resType count

![res_type](https://www.github.com/Byzero512/blog_img/raw/master/1537247141180.png)

> such as: bVar resb 10 // bVar 在 bss 段有10个db的空间

![enter description here](https://www.github.com/Byzero512/blog_img/raw/master/1537247177276.png)

4. extern 变量声明

### 数据单元的引用
1. byte [var_name] ------- byte ptr [var_name]  //方括号其实可以省略
2. word [var_name] ------- word ptr [var_name]
3. dword [var_name] ------- dword ptr [var_name]
4. qword [var_name] ------- qword ptr [var_name]
> such as:
> mov al byte [var_name]

### 节
> such as:
> section .text
> section .data
> section .bss

### 简单的框架
```x86asm

section .data
bVar db 10;   bvar=10

section .bss
bVar_uninit resb 1; 声明1个byte的空间给变量 bVar_uninit

section .text

;如果要使用标准的系统连接器,那么程序入口点应该这样子声明:
global _start 
_start:          

<code here>

;终止程序其实不需要标签,即 last: 可以省略

last:

<code here>

```

## 指令语法

### 前置
1. 两个操作数应该具有相同的大小

### mov
>  两个操作数不能同时为内存

``` x86asm
mov rax,100
mov rax qword [var1]; 将var1的值放进rax中
mov rax var1; 将var1的地址放进rax中
```
#### 注意

1.  for double-word destination and source operand, the upper-order portion of the quadword register is set to 0.

``` x86asm
mov eax,100;
mov rcx,-1;
mov ecx,eax;//这一步之后rcx=0,但如果是 mov cx,ax 或者 mov cl,al,那么只有cx或者cl被改变
```



### lea
> 获得一个地址

```x86asm
lea rsi,byte[var]; 将var的地址放进rsi中,那么 rsi中存放的就是 byte_ptr
lea rsi,dword [var];
```

### 类型转换
> 在汇编语言中可以通过改变要操作的操作数的大小类实现类型转换
> 如: mov rax,byte[var1] 和 mov rax,word[var1]

#### narrowing conversions


#### widening conversions

