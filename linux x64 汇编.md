---
title: linux x64 汇编
description: 这篇文章是读<x64 Assembly Language Programming with Ubuntu>的读书笔记,简单地记录了一些x64汇编指令,包括浮点数指令.
author: byzero(or named Byzero512) 
categories: 
- asm
- x86-64 asm
tags: 
- asm
- x86-64 asm
---

## Preface

### 汇编器
> 本文使用 ysam 汇编器
> yasm -g dwarf2 -f elf64 example.asm -l example.lst -o output_file.o

> nasm -g -f elf64 suorce.asm -l list_file -o output_file.o

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

## 符号语法

### 符号声明

#### const 常量声明

> var_name  equ  var_value
> such as: SIZE equ 1000

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

#### 函数声明
> 函数声明应该在 .text 中
```x86asm
global <procName>
<procName>:
	;function body
ret
```

#### extern 符号声明
> 形式: extern \<symbolName>

### 数据单元的引用
> 在写汇编时,用前面的写法,后面的写法汇编器编译时会认为是个语法错误 (但是反汇编时,又一般以后面的写法表示)

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
SYS_exit equ 60

section .bss
bVar_uninit resb 1; 声明1个byte的空间给变量 bVar_uninit

section .text

;如果要使用标准的系统链接器,那么程序入口点应该这样子声明:
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
> 汇编中,整体上来说只有两种指令: 与整数有关的指令,与浮点数有关的指令

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
> 在汇编语言中可以通过改变要操作的操作数的大小来实现类型转换
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

![add_1](https://www.github.com/Byzero512/blog_img/raw/master/1537365975136.png)

![adc_1](https://www.github.com/Byzero512/blog_img/raw/master/1537366001680.png)

2. #### 减法

> sub
> dec \<op> 减一
> sbb

![sub_1](https://www.github.com/Byzero512/blog_img/raw/master/1537365940910.png)

3. #### 乘法

> multiplying two n-bit values produces a 2n-bit result at most

+ unnsigned multiplication: mul
> 操作数不能是立即数
> mul \<op>, 根据 op 的大小,另一个数放在了 al,ax,eax,rax 中

![example](https://www.github.com/Byzero512/blog_img/raw/master/1537336933113.png)

![mul_result](https://www.github.com/Byzero512/blog_img/raw/master/1537336613629.png)

+ signed multiplication: imul
> 如果自定义\<dest>,那么 \<dest> 必须是 \<reg>
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

### 浮点数
> The text focuses on the x86-64 floating-point operations, which are not the same as th32-bit floating-point operatio
> 浮点数的运算指令中,两个操作数必须是同一类型

#### xmm register: 浮点数寄存器
> xmm0--xmm15
> xmm寄存器在新的cpu中为 128 bytes 或者 256 bytes,
> xmm寄存器不仅用于浮点数的处理,还可以用于对图像的处理,其实后者才是xmm寄存器的真正用途

#### Data Movement: 浮点数的移动
> movss \<dest>,\<src>, 对 float 的操作
> movsd \<dest>,\<src>, 对 double 的操作

![float_1](https://www.github.com/Byzero512/blog_img/raw/master/1537364909387.png)

![float_2](https://www.github.com/Byzero512/blog_img/raw/master/1537364934560.png)

#### Integer / Floating-Point Conversion Instructions: 整数/浮点数的转换

> 如果整数需要参加浮点数的运算, 那么整数必须转换成浮点数

![float_int_conversion_1](https://www.github.com/Byzero512/blog_img/raw/master/1537365324558.png)

![float_int_converrsion_2](https://www.github.com/Byzero512/blog_img/raw/master/1537365300788.png)

#### Floating-Point Arithmetic Instructions: 浮点数运算指令之加减乘除

> 浮点数的运算指令中,两个操作数必须是同一类型
> **\<dest> 必须是浮点数寄存器**
> **\<src> 不能是立即数**

1. ##### 加法

> addss \<RXdest>,\<src>
> addsd \<RXdest>,\<src>
> **\<dest> 必须是浮点数寄存器**
> **\<src> 不能是立即数**

![add_float](https://www.github.com/Byzero512/blog_img/raw/master/1537366073136.png)

2. ##### 减法

> subss \<RXdest>,\<src>
> subsd \<RXdest>,\<src>
> **\<dest> 必须是浮点数寄存器**
> **\<src> 不能是立即数**

![sub_float](https://www.github.com/Byzero512/blog_img/raw/master/1537366577803.png)

3. ##### 乘法

> mulss \<RXdest>,\<src>
> mulsd \<RXdest>,\<src>
> **\<dest> 必须是浮点数寄存器**
> **\<src> 不能是立即数**

![mul_float](https://www.github.com/Byzero512/blog_img/raw/master/1537366894266.png)

4. ##### 除法

> divss \<RXdest>,\<src>
> divsd \<RXdest>,\<src>
> **\<dest> 必须是浮点数寄存器**
> **\<src> 不能是立即数**

![div_float](https://www.github.com/Byzero512/blog_img/raw/master/1537367018033.png)

5. ##### 平方根

> sqrtss \<RXdest>,\<src>
> sqrtsd \<RXdest>,\<src>
> **\<dest> 必须是浮点数寄存器**
> **\<src> 不能是立即数**

![sqrt_float_1](https://www.github.com/Byzero512/blog_img/raw/master/1537367149782.png)

![sqrt_float_2](https://www.github.com/Byzero512/blog_img/raw/master/1537367169651.png)

#### Floating-Point Comparison: 浮点数比较指令
> ucomiss \<RXsrc>,\<src>
> ucomisd \<RXsrc>,\<src>
>  **\<RXsrc> 必须是浮点数寄存器**
> **\<src> 不能是立即数**

![cmp_float](https://www.github.com/Byzero512/blog_img/raw/master/1537367794787.png)

![cmp_float_1](https://www.github.com/Byzero512/blog_img/raw/master/1537367876109.png)

#### Floating-Point Control Instructions: 浮点数控制指令
> Floating-Point Comparison
> float-conditional-jmp

#### Floating-Point Calling Conventions: 浮点数调用约定
> xmm0--xmm7

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
mov rax,qword ptr [Var1]; 和上面的指令意义一样,在写汇编时,应该用上一种方式,这种方式一般是在反汇编时才能看到,不能这么写(反正我用下面这种方法,汇编器说这是个语法错误)

;上述三条指令,由于访问的是变量,故机器码中存放的是变量地址,而不是变量的值
```

## stack implementation
+ 汇编语言中其实本身并没有局部变量声明这个东西
	+ 减小 rsp 来创建所谓的局部变量
	+ 根据 rbp偏移 来访问所谓的局部变量
> 在某些汇编器中,可以使用伪代码来创建所谓的局部变量

> push
> pop
> use register rip and rbp

## macros
> 宏定义应该被放在 .data 和 .text 之前

### single-line macros

```x86asm
%define Macro_name shl rax,2;

%define Macro_name(x) shl x,2;

```

### multi-line macros

#### define_format
```x86asm
%macro <name> <number of arguments>

;body of macro

%endmacro
```

#### example
```x86asm
;--------------define macro------------

%macro abs 1
	cmp %1,0            ;%1代表了第一个参数
	jge %%done
	neg %1
	
%%done:         ; 宏定义中的label前面必须有2个'%'
%endmarco

;-----------use macro----------
mov eax,1
abs eax

```

## Functions

+ 函数有两个特性
	+ linkage: 可以在程序不同的地方调用并且正确返回
	+ argument transmission: 可以访问参数并且返回值

### function declaration: 函数声明
> 不能在函数声明中声明函数

```x86asm
global <procName>
<procName>:
	;function body
ret
```

### Standard Calling Convention: 函数调用约定
> 参数传递
> 返回值
> 系统调用参数

#### 函数参数

1. 整数或指针

![function_arg](https://www.github.com/Byzero512/blog_img/raw/master/1537348017268.png)

2. 浮点数
> xmm0-xmm7

#### 函数返回值
> 根据大小使用 A 寄存器 或者 xmm0 寄存器

#### 寄存器在函数调用时的作用

![call_1](https://www.github.com/Byzero512/blog_img/raw/master/1537348351834.png)

![call_2](https://www.github.com/Byzero512/blog_img/raw/master/1537348365400.png)

> 一个调用约定的链接
https://en.wikipedia.org/wiki/X86_calling_conventions#System_V_AMD64_ABI


### invoke: 调用函数

```x86asm
push argument
call <function>
```

### Red Zone
> 当我们调用函数,保存了寄存器的状态后,会有一部分的区域给编译器进行局部变量的优化

![red_zone](https://www.github.com/Byzero512/blog_img/raw/master/1537348712514.png)


## 与操作系统的交互: syscall
> 系统调用号: rax
> 参数: rdi,rsi,rdx,rcx,r8,r9


## external symbol: 外部符号引用
>  In general, using global variables accessed across multiple files is considered poor programming practice and should be used sparingly (if at all)

```x86asm
extern <symbolName>
```

## 命令行参数
> 操作系统负责命令行参数的解析或者读取
> 命令行参数在运行 \_start 前, 根据调用约定被放进了对应的寄存器中,所以可以通过操作对应寄存器来处理命令行参数

## buffer: 缓冲区
> 在程序运行期间,由于访问 secondary storage 的开销远远大于访问 main memory,所以为了提高效率,需要限制访问 secondary storage 的次数.
> 缓冲区是一个临时存储从二级存储设备获得的数据的内存段
> 也可以是临时存储要输出到二级存储设备的数据的内存段

---

## Parallel Processing: 并行处理
+ 进程并行
	1. 几个进程处在不同的 core 中: 真并行
	2. 几个进程处在一个相同的 core 中: 通过算法和调度实现的假并行

> 进程并行可能是多个进程完成多个任务,也可能是共同完成单个任务

### 实现进程并行的方式
> The basic approaches to parallel processing are distributed computing(分布式计算) and multiprocessing(also referred to as threaded computations)
> 不管是哪种方法,都要实现共享内存多处理

#### Distributed computing: 分布式计算

> 分布式运算可以认为是把一个大问题分解成很多小问题, 分布式系统中每一个计算机都去处理一个小问题, 然后综合计算结果

#### Multiprocessing: 多道处理
> 现在计算机的CPU一般是多核的, 每一个 core 对内存资源都有相同的访问权限
> A thread is often referred to as a light-weight process since it will use the initial process’s allocations and address space thus making it quicker to create. Since the threads share memory with the initial process and any other threads, this presents the potential for very fast communication between simultaneously executing threads.
> 但是这种方法需要保证结果不被 corrupted, 即要注意 race condition
> 其次,计算机的核数有限

##### POSIX Threads
> The POSIX Threads, commonly called pThreads, is a widely available thread library on Ubuntu and many other operating systems. 
> Ideally, in order to maximize the overall parallel operations, the main process would perform other computations while the thread or threads are executing and only check for thread completion when the other work has been completed
> 线程之间防止竞争,实现同步

##### race condition: 条件竞争
> 条件竞争指的是多个线程同时对同一个数据进行修改


## Interrupts: 中断
> Typically, the current process is interrupted so that some other work can be performed. An interrupt is usually defined as an event that alters the sequence of instructions executed by a processor. Such events correspond to signals generated by software and/or hardware. 


> 中断有不同的类别和优先级
> 中断是操作系统提高资源利用率的主要方式

### 类别
1. 软件中断和硬件中断
> software interrupt
> + Software programs can also generate interrupts to initiate I/O as needed, request OS services, or handle unexpected conditions.	

> hardware interrupt
> + Handling interrupts is a sensitive task. Interrupts can occur at any time, the kernel triesto get the interrupt addressed as soon as possible. 

> Additionally, an interrupt can beinterrupted by another interrup
> cpu在执行某些指令时是不能被中断的

2. 同步中断和异步中断
> Synchronous Interrupts: 同步中断
> + 在CPU的控制下产生的中断或者由进程造成
> + 同步中断的性质和中断产生的位置有关

> Asynchronous Interrupts: 异步中断
> + 任意时刻都可以发生,并且不能预测
> + 可以使外部硬件导致的中断

3. 可屏蔽中断和不可屏蔽中断
> Maskable interrupts
> + 可屏蔽中断表示中断可以延迟处理

> Non-maskable interrupts
> + 不可屏蔽中断表示中断必须马上处理,一般由CPU处理

### interrupt Privilege Levels: 中断优先级
> Privilege Levels refer to the privilege level at which the interrupt code executes(一般来说)

#### code execute level

![enter description here](https://www.github.com/Byzero512/blog_img/raw/master/1537426965775.png)

### 中断处理
1. 中断发生
2. 挂起
3. 保存状态: rip, rflags ...
4. Interrupt Descriptor Table: 中断处理向量表
	+ The start of the IDT is pointed to by a dedicated register, IDTR, which is only accessible by the OS and requires level 0 privilege to access
5.  Interrupt Service Routine: 中断处理例程
6.  iret, 恢复

![int_handle](https://www.github.com/Byzero512/blog_img/raw/master/1537428330526.png)

> The steps are details as follows:
1. Execution of current program is suspended
- save rip and rFlags registers to stack
2. Obtain starting address of Interrupt Service Routine (ISR)
- interrupt number multiplied by 16
- used as offset into Interrupt Descriptor Table (IDT)
- obtains ISR address (for that interrupt) from IDT
3. Jump to Interrupt Service Routine
- set rip to address from IDT
4. Interrupt Service Routine Executes
- save context (i.e., any additional registers altered)
- process interrupt (specific to interrupt generated)
- schedule any later data processing activities
5. Interrupted Process Resumption
- resume scheduling based on OS scheduler
- restore context
- perform iret (to pop rFlags and rip registers)
> This interrupt processing mechanism allows a dynamic, run-time lookup for the ISR
address
