---
title: pwnExp house-of series 
description: 
categories:
tags: 
---

### house of spriit: 伪造fastchunk
> 伪造fast chunk, 并链接在fastbin上或者通过malloc_consolidate链接在smallbin上面

1. uaf
2. unlink(一般和smallbin或者unsortedbin的攻击一起用,而且一般unlink的chunk为fake的小chunk)


### house of lore: 攻击smallbin
> 相比于攻击smallbin,攻击unsortedbin要简单一些
> 伪造好 small_chunk 放在 smallbin上面面

#### 攻击前提
> 要能设置好victim,以及Bck(Bck=victim->bk)的fd指针



### house of force: 攻击topchunk
> 攻击 top chunk
> 原理是整数溢出造成的任意读写: 如 leak libc,改got 

#### 攻击前提
1. 知道 topchunk 的地址
2. 知道 target 的地址

#### 攻击流程
1. 把 top_chunk 的size改的很大
> 如 -1

2. 然后malloc一个非常大的chunk
> size= target_adr-4\*8-topchunk_ptr
> malloc(size)

3. 由于整数溢出,结果是 原本指向topchunk的ptr指向了bss段之类的位置
> topchunk_ptr = top_chunk_ptr + { target_adr - 4\*8 - topchunk_ptr } + 2\*8
> topchunk_ptr = top_chunk_ptr - 2\*8

4. malloc 一个chunk,此时这个chunk的 usr_ptr=target_adr

> 总的来说,是整数溢出导致的任意写


### house of einherjar: 攻击 top_chunk
> 这里的攻击top_chunk和house of force 刚好反过来了
> **house of force 是 malloc top_chunk**
> **house of einherjar 是 合并 top_chunk**
> house of einherjar先通过一些漏洞改变与topchunk相邻的chunk(chunk_near_top)的prev_size和prev_inuse为特定值,然后free掉这个和 topchunk 相邻的 chunk, top_chunk就会合并前两个chunk,然后就可以在某个预期的地址分配出chunk.

+ 设置好和topchunk相邻的那个chunk的相关信息
1. 改变prev_size
> prev_size=chunk_near_top_adr-target_adr

2. 改变prev_inuse
> prev_inuse=0 (可以通过 null byte off by one 改变, 也可以通过 free 前一个chunk (注意不是fastchunk) 改变)

3. 设置好目标地址的 fake_chunk 的相关字段
> fake_chunk会被认为是一个 freed large chunk, 所以还要设置 nextsize 链
> 要绕过 unlink 检查, 而且prev_inuse=1(这里要比smallchunk unlink绕过要麻烦)

### house of orange
> 实际上是利用malloc的时候如果topchunk小于请求的大小, 会把 topchunk 挂载到 unsortedbin上, 从而实现free的效果


### house of rabbit
> 实际上是利用malloc_consolidate的时候没有检查fastbin的大小.

