---
title: 2018-hitconctf writeup
tags: 
categorires: 
author: 
top: 
---


## baby tcache
> 这道题可以最容易想到的就是使用 house of Roman
> 但是, 在使用 tcache 这种检查稀少的机制中, 出现了新的攻击方式, 使用 stdout iofile 里的指针进行泄露信息

### 题目流程
> 菜单题, 但是 noleak, noedit
> 1. new
> 2. delete

### 漏洞点
> null off by one

### 利用过程
#### leak libc
1. null off by one 构造overlap.
	1. 小的chunk能进入tcache
	2. 大的chunk需要能进入unsortedbin
2. 让小的chunk中fd的位置出现libc里的指针.
	1. malloc unsortedbin中的 chunk, 大小要是小chunk刚好链在unsortedbin上
3. free掉第二步malloc出来的chunk, 会合并到 unsortedbin中剩下的chunk中
4. unsortedbin 中的 chunk 全部 malloc 出来
	1. 对小chunk进行大小的修复, 
	2. 爆破小chunk的fd的低 2 bytes, 使其指向stdout的 \_IO_write_ptr
4. 经过上述的步骤, tcache中应该有指向stdout的chunk, malloc出来
	1. 将指向stdout的_IO_write_ptr的chunkmalloc出来, 并且修改_IO_write_ptr(原本就指向libc中的某个地址)的低位为 0xfff8
5. 这样子, 调用puts()函数的时候, 会输出很多信息, 其中有libc中的地址, 这样子就leak libc了

> 上面的方法, 只需要爆破4bit就可以得到libc, 但是比较特殊, 需要用到tcache机制, 在低版本的libc不能使用

#### 写 malloc_hook 为 onegadget
> overlap, malloc一个chunk到malloc_hook, 写入onegadget


### something
> 本题是对stdout缓冲区的攻击


