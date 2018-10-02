---
title: pwnExp house-of series 
description: 
categories:
tags: 
---

### house of spriit: 伪造fastchunk
> 伪造fast chunk, 并链接在fastbin上或者通过malloc_consolidate链接在smallbin上面

1. uaf
2. unlink(一般和smallbin或者unsortedbin的攻击一起用)

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

3. 结果是 原本指向topchunk的ptr指向了bss段之类的位置
> topchunk_ptr = top_chunk_ptr+{target_adr-4\*8-topchunk_ptr}+2\*8 = target_adr-2\*8

4. malloc 一个chunk,此时这个chunk的 usr_ptr=target_adr

> 总的来说,是整数溢出导致的任意写

### house of lore: 攻击smallbin
> 相比于攻击smallbin,攻击unsortedbin要简单一些

#### 攻击前提
> 要能设置好victim,以及Bck(Bck=victim->bk)的fd指针

