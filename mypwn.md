---
title: mypwn
tags: 
categorires: 
author: 
top: 
---


## 结构
- mypwn
	- fmt: 拓展pwntools, 可以生成amd64下的fmtstr payload
		- fmt_payload()
		- fmt_offset()
	- iofile: 自动生成iofile攻击的payload
		- fsop()
	- patch: 自动patch
		- exchangeGot()
	- submit_flag: 自动提交flag
		- class cookies
	- shell: 自动生成一些shellcode
		- 暂时没有

## fmt
### fmt_payload(): 返回格式化字符串攻击的payload
> fmt_payload(offset,writes={},just_change_low_bit=1,show_payload)

> 参数解析
> 1. offset: 为格式化字符串的offset
> 2. writes: 提供 {addr:value}
> 3. just_change_low_bit: 如果设置为1, 只会根据writes={}提供的键值对, 修改对应地址的低位, 否则会用 "\x00" 补全, 修改完整的 8 bytes 或者 4 bytes

### fmt_offset(): 返回格式化字符串攻击的offset
> fmt_offset(fmtstr_posi,esp_posi)

> 参数解析
> 1. fmtstr_posi: 格式化字符串在内存(栈)的位置
> 2. esp_posi: 寄存器ESP的值

## iofile
### fsop(): 生成iofile攻击中修改vtable位置的payload
> fsop_payload(iofile_addr,writes={},ptr_to_zero=0x0)

> 参数解析
> 1. iofile_addr: iofile结构体在内存的地址
> 2. writes={}: {'read':addr} 或者 {'puts'|'write':addr} 或者 {'finish':addr} 或者 {'overflow':addr}
> 3. ptr_to_zero=0: 一个指向为0的区域的指针, 以设置结构体中的lock的位置, 如果不提供会自动生成


## patch
### exchangeGot(): 交换文件函数got表的位置, 以达到防御的目的
> exchangeGot(fpath)

> 参数解析
> 1. fpath: 二进制文件的位置



