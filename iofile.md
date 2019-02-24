---
title: iofile
tags: 
categorires: 
author: 
top: 
---


## fread

1. 先读缓冲区 read_ptr -- read_end
2. 读完缓冲区, 从文件读:
	+ 要读的数据小于缓冲区: 读到 buf_base--buf_end
	+ 要读的数据大于缓冲区: 缓冲区大小的整数倍的数据直接到目的地址, 剩下的按小于缓冲区的方式输入

### 一致
1. 如果要使用到缓冲区, 
	+ 进行读系统调用之前, 需要将 read缓冲区, write缓冲区的指针全部设置为 buf_base
	+ 读完之后, 设置 read_end

### 利用
flag:
	1. \~\_IO_IN_BACKUP (这个必须设置)
	2. \~\_IO_CURRENTLY_PUTTING (尽量设置, 否则应该保证在读之前不需要输出,即write_ptr<write_base; )
	3. \~\_IO_NO_READS (这个必须设置)
	
ptr:
	1. read_ptr == read_end (读缓冲区空了)
	2. buf_end - buf_base > 要读入的数据
 	3. buf_base--buf_end 包含要读入的目的地址
	4. write指针尽量置空或者相等(保证write_ptr<=write_base)


## fwrite
1. 先填满缓冲区(一般而言write_end和和buf_end是样的)
2. 输出缓冲区的数据
3. 缓冲区整数倍输出
4. 将剩下数据填入缓冲区

### 一致
1. 不管是不是用缓冲区, 每一次写完之后
	1. read缓冲区全部被设置完 buf_base
	2. write_base write_ptr 被设置为 buf_base
	2. 如果是行缓冲或者无缓冲, write_end 被设置为 buf_base, 如果是满缓冲, write_end 被设置为 buf_end

### 利用
flag:
	1. \_IO_CURRENTLY_PUTTING
	2. \_IO_LINE_BUF(可能可以有可以没有)

ptr:
	write_base -- write_ptr 包含要输出的目的地址
	read_end == write_base (这个必须设置, 否则会seek位置)