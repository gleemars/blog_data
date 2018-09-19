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

#### const 常量声明

> var_name  equ  var_value
> such as: SIZE equ 1000;

#### 有初始化的变量声明

> 形式: var_name  dataType  var_value

![init_var_type_1](https://www.github.com/Byzero512/blog_img/raw/master/1537247003007.png)

![init_var_type_2](https://www.github.com/Byzero512/blog_img/raw/master/1537247014703.png)

> such as:  
> bVar db 10 //这么声明是一个整数
> bChar db "1" //这么声明是一个字符
> bStr db "10" //这么声明是一个字符串

![define_init_var_example](https://www.github.com/Byzero512/blog_img/raw/master/1537246736243.png)

#### 没有初始化的变量声明

> 形式: var_name resType count

![res_type](https://www.github.com/Byzero512/blog_img/raw/master/1537247141180.png)

> such as: bVar resb 10 // bVar 在 bss 段有10个db的空间

![enter description here](https://www.github.com/Byzero512/blog_img/raw/master/1537247177276.png)

#### extern 变量声明??????????????

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
EXIT_SUCCESS equ 0
SYS_exit que 60

section .bss
bVar_uninit resb 1; 声明1个byte的空间给变量 bVar_uninit

section .text

;如果要使用标准的系统连接器,那么程序入口点应该这样子声明:
global _start 
_start:          

<code here>

;终止程序其实不需要标签,即 last: 可以省略

last:
mov rax,SYS_exit
mov rdi,SYS_SUCCESS
syscall;	exit(0)

<code here>

```

## 基本指令 I

### 前置
> 在计算指令中,两个操作数必须具有相同的长度

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
---

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
> 控制好 src 的大小就可以了

#### widening conversions
1. unsigned conversions
> movzx \<dest> \<src> , 会把高位置零
> 但是不支持 movzx \<reg64>,\<op32> 这种转换,但是可以通过mov指令实现

![unsigned_converrsion](https://www.github.com/Byzero512/blog_img/raw/master/1537332829261.png)


2. signed conversions
> 由于符号的存在, 要把高位的数字都设置为符号位
> movsx \<dest>,\<src>, 不支持32位到64位的转换
> movsxd \<dest>,\<src>, 只由于32位到64位的有符号转换

![signed_conversion_1](https://www.github.com/Byzero512/blog_img/raw/master/1537342256544.png)

![signed_conversion_2](https://www.github.com/Byzero512/blog_img/raw/master/1537342268713.png)

### 整数指令:加减乘除

1. #### 加法

> add 不检查 rflag 的 CF 
> inc \<op> 加一
> adc 检查进位的加法指令,一般用于大数字的加法

2. #### 减法

> sub
> dec \<op> 减一
> sbb

3. #### 乘法

> multiplying two n-bit values produces a 2n-bit result at most

+ unnsigned multiplication: mul
> 操作数不能是立即数
> mul \<op>, 根据 op 的大小,另一个数放在了 al,ax,eax,rax 中

![example](https://www.github.com/Byzero512/blog_img/raw/master/1537336933113.png)

![mul_result](https://www.github.com/Byzero512/blog_img/raw/master/1537336613629.png)

+ signed multiplication: imul
> 如果自定义\<dest>,那么 dest 必须是 reg
> For the multiple operand multiply instruction, byte operands are not supported
> 程序员要根据op的大小选择对应的指令

![instruction](https://www.github.com/Byzero512/blog_img/raw/master/1537337053189.png)
``` x86asm
imul <source>
;same as mul,but the op is signed

imul <dest>,<src/imm>
;dest=dest*src/imm
;A byte size destination operand is not supported

imul <dest>,<src>,<imm>
;dest=src*imm ,the <src> operand must be a register or memory location
;A byte sized destination operand is not supported.
```
![imul_example](https://www.github.com/Byzero512/blog_img/raw/master/1537338289942.png)

4. ####  除法
>  dividend must be a larger size than the divisor
> unsigned division: div \<op>
> signed division: idiv \<op>

![division_op](https://www.github.com/Byzero512/blog_img/raw/master/1537338694315.png)

![division_layout](https://www.github.com/Byzero512/blog_img/raw/master/1537338735154.png)

![enter description here](https://www.github.com/Byzero512/blog_img/raw/master/1537338821935.png)

### 逻辑指令

> 操作数要具有相同大小
> and
> or
> xor
> not 操作数不能是立即数

### Shift Operations

#### logical shift

> The logical shift treats the operand as a sequence of bits rather than as a number.

![logic_shfit](https://www.github.com/Byzero512/blog_img/raw/master/1537339666352.png)

#### Arithmetic Shift
> 算术左移不保留符号位,可以用作 乘2 的快速乘法,其实和shl意义一样
> 算术右移可以看作是符号位的拓展
+ The arithmetic shift right is also a bitwise operation that shifts all the bits of its source register by the specified number of bits places the result into the destination register. 
+  For an arithmetic left shift, the  original leftmost bit (the sign bit) is replicated to fill in all the vacant positions.

![shift_right](https://www.github.com/Byzero512/blog_img/raw/master/1537340351822.png)

![sal_sar](https://www.github.com/Byzero512/blog_img/raw/master/1537340250733.png)

#### roate operations shift
> 旋转操作数

![roate_shift](https://www.github.com/Byzero512/blog_img/raw/master/1537340852936.png)

## 基本指令 II

### label 定义标签

![label_def](https://www.github.com/Byzero512/blog_img/raw/master/1537341502896.png)

### jmp

#### unconditional control innstructions: jmp,距离无限制跳转

![uncon_jmp](https://www.github.com/Byzero512/blog_img/raw/master/1537341810033.png)

#### conditional control instructions: 短转移,限制为 ±128 bytes

> 需要两步:

 1. cmp 设置 rflag 的 psw

![cmp](https://www.github.com/Byzero512/blog_img/raw/master/1537341910342.png)

 2. 执行相应的有条件jmp语句

![jmp_conditional](https://www.github.com/Byzero512/blog_img/raw/master/1537341928610.png)

### Iteration 迭代/循环: loop
> 在汇编循环中,一般使用 rcx 作为 counter
> loop \<label>, 下图左边和右边的意义一样
> loop时,会判断 rcx==0,如果不成立,会执行跳转执行

![loop_label](https://www.github.com/Byzero512/blog_img/raw/master/1537343163502.png)

举个简单地例子: 奇数求和

1. before use loop:

![sum_1](https://www.github.com/Byzero512/blog_img/raw/master/1537342992645.png)

2. use loop: 

![sum_2](https://www.github.com/Byzero512/blog_img/raw/master/1537343348265.png)


## addressing modes: 寻址方式
> This chapter provides some basic information regarding addressing modes and the associated address manipulations on the x86-64 architecture

+ The basic addressing modes are:
	+ 1. Register
	+ 2. Immediate
	+ 3. Memory

### address and value

```x86asm
mov rax,Var1;	rax=&Var1

mov rax,qword [Var1];	rax=Var1,其实这条指令的机器码中放的是Var1这个变量的地址,但是取的是值,而不是地址

mov rax,qword ptr [Var1];	rax=*Var1

;上述三条指令,由于访问的是变量,故机器码中存放的是变量地址,而不是变量的值
;但由于指令码不同,操作就不同.一个地址,一个根据地址取值,一个取值当成地址再取值
```
