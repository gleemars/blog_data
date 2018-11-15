---
title: mips语法整理
tags: 
categorires: 
author: 
top: 
---

## 寄存器
1. $v0 返回值
2. $sp 栈顶指针寄存器
3. $fp 栈帧基址寄存器
4. $ra 函数返回地址

> $0 永远为0
> $a0 -- $a3 参数寄存器
> $t0 -- $t9   临时寄存器
> $s0 -- $s7 (进入子函数)可能需要保存的寄存器

> 还有其他寄存器不介绍


## 汇编语言

### 变量声明


### 






## 系统调用
+ 调用号: $v0
+ 参数: $a0 -- ??
+ 系统调用指令: syscall


## 函数
1. 保存环境(如果需要的话)
> $t0 - $t9 由调用者保存

2. 设置参数
> $a0 -- $a3; 多余的放栈上

3. 跳转
> jal jalr
> bal

4. 进入子函数, 创建栈帧, 保存上下文



## 指令
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