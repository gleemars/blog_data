---
title: _IO_FILE 结构与利用
description: 
categories:
tags: 
grammar_cjkRuby: true
---

### 前置
1. 打开一个文件时,会在堆上创建一个 locked_FILE 结构,并返回一个FILE类型指针

### \_IO_list_all
> 一个记录了所有 \_IO_FILE_plus 结构的单向链表的链表头,储存在libc中

### FILE
1. \_IO_FILE
> 描述文件的一个结构体

```cpp
struct _IO_FILE {
  int _flags;		/* High-order word is _IO_MAGIC; rest is flags. */
#define _IO_file_flags _flags

  /* The following pointers correspond to the C++ streambuf protocol. */
  /* Note:  Tk uses the _IO_read_ptr and _IO_read_end fields directly. */
  
  /*===================================================
  记录读操作发生的区域: [ _IO_read_base : _IO_read_ptr ]
  ===================================================*/
  char* _IO_read_ptr;	/* Current read pointer */                //0x10 0x8
  char* _IO_read_end;	/* End of get area. */                     //标记文件内容在缓冲区的区域: [ _IO_buf_base : _IO_read_end ]
  char* _IO_read_base;	/* Start of putback+get area. */       
  
  /*===================================================
  记录写操作发生的区域: [ _IO_write_base : _IO_write_ptr ]
  ===================================================*/
  char* _IO_write_base;	/* Start of put area. */                   //0x20 0x18
  char* _IO_write_ptr;	/* Current put pointer. */                 //0x28  0x1c
  char* _IO_write_end;	/* End of put area. */
  
  /*===================================================
  记录文件的缓冲区位置: [ _IO_buf_base : _IO_buf_end ]
  ===================================================*/
  char* _IO_buf_base;	/* Start of reserve area. */           
  char* _IO_buf_end;	/* End of reserve area. */             
  /* The following fields are used to support backing up and undo. */
  
  char *_IO_save_base; /* Pointer to start of non-current get area. */
  char *_IO_backup_base;  /* Pointer to first valid character of backup area */
  char *_IO_save_end; /* Pointer to end of non-current get area. */

  struct _IO_marker *_markers;

  struct _IO_FILE *_chain;

  int _fileno;                               //文件描述符, 唯一修改这个来伪造stdin,stdout
#if 0
  int _blksize;
#else
  int _flags2;
#endif
  _IO_off_t _old_offset; /* This used to be _offset but it's too small.  *///14

#define __HAVE_COLUMN /* temporary */
  /* 1+column number of pbase(); 0 is unknown. */
  unsigned short _cur_column;
  signed char _vtable_offset;
  char _shortbuf[1];//0x88 do what?

  /*  char* _save_gptr;  char* _save_egptr; */

  _IO_lock_t *_lock;
#ifdef _IO_USE_OLD_IO_FILE
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
3. \_IO_FILE_complete
```cpp
struct _IO_FILE_complete
{
  struct _IO_FILE _file;
#endif
#if defined _G_IO_IO_FILE_VERSION && _G_IO_IO_FILE_VERSION == 0x20001
  _IO_off64_t _offset;
# if defined _LIBC || defined _GLIBCPP_USE_WCHAR_T
  /* Wide character stream stuff.  */
  struct _IO_codecvt *_codecvt;
  struct _IO_wide_data *_wide_data;
  struct _IO_FILE *_freeres_list;
  void *_freeres_buf;
# else
  void *__pad1;
  void *__pad2;
  void *__pad3;
  void *__pad4;
# endif
  size_t __pad5;
  int _mode;                                                   //这个是打开模式
  /* Make sure we don't get into trouble again.  */
  char _unused2[15 * sizeof (int) - 4 * sizeof (void *) - sizeof (size_t)];
#endif
};
```

4.  \_IO_FILE_plus
> 封装了上面的结构

```cpp
struct _IO_FILE_plus
{
  _IO_FILE file;
  const struct _IO_jump_t *vtable; // 64位偏移为 0xd8, 32位下偏移为0x94
};
```

5. \_IO_wide_data
```cpp
struct _IO_wide_data
{
wchar_t *_IO_read_ptr; /* Current read pointer */
wchar_t *_IO_read_end; /* End of get area. */
wchar_t *_IO_read_base; /* Start of putback+get area. */

wchar_t *_IO_write_base; /* Start of put area. */
wchar_t *_IO_write_ptr; /* Current put pointer. */
wchar_t *_IO_write_end; /* End of put area. */

wchar_t *_IO_buf_base; /* Start of reserve area. */
wchar_t *_IO_buf_end; /* End of reserve area. */
/* The following fields are used to support backing up and und
o. */
wchar_t *_IO_save_base; /* Pointer to start of non-current
get area. */
wchar_t *_IO_backup_base; /* Pointer to first valid charact
er of
backup area */
wchar_t *_IO_save_end; /* Pointer to end of non-current get
area. */
__mbstate_t _IO_state;
__mbstate_t _IO_last_state;
struct _IO_codecvt _codecvt;
wchar_t _shortbuf[1];
const struct _IO_jump_t *_wide_vtable;
};
```

6. locked_FILE
> 实际上在堆上的是这个结构体
> struct locked_FILE \*new_f = (struct locked_FILE \*) malloc (sizeof (structt locked_FILE));
```cpp
struct locked_FILE
{
struct _IO_FILE_plus fp;
#ifdef _IO_MTSAFE_IO
_IO_lock_t lock;           
#endif
struct _IO_wide_data wd;
};
```

## 文件IO操作
> 从IO库函数 --> FILE文件结构 --> 文件结构vtable中对应的函数 --> 系统调用
> 所有的文件IO操作, 第一个参数都是一个指向_IO_FILE_plus的指针

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
>  调用vtable中的函数指针指向的函数时, 函数的第一个参数都是一个指向_IO_FILE_pllus 的指针, 所以需要向\_IO_FILE_plus的起始处写入"/bin/sh\x00"

### 利用方式
> 1. exploit vtable

### fake \_IO_FILE_plus
1. 成员偏移
```cpp
0x0   _flags
0x8   _IO_read_ptr
0x10  _IO_read_end
0x18  _IO_read_base
0x20  _IO_write_base
0x28  _IO_write_ptr
0x30  _IO_write_end
0x38  _IO_buf_base
0x40  _IO_buf_end
0x48  _IO_save_base
0x50  _IO_backup_base
0x58  _IO_save_end
0x60  _markers
0x68  **_chain**
0x70  **_fileno**
0x74  _flags2
0x78  _old_offset
0x80  _cur_column
0x82  _vtable_offset
0x83  _shortbuf
0x88  **_lock**
0x90  _offset
0x98  _codecvt
0xa0  _wide_data
0xa8  _freeres_list
0xb0  _freeres_buf
0xb8  __pad5
0xc0  **_mode**
0xc4  _unused2
0xd8  **vtable**
```
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
+ 局限性
	1. 从libc2.24开始,由于新增加了对vtable的检查,伪造vtable就不能用了

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
fake_file += p64(buf_addr + 0x10 - 0x88)        # fake_vtable_addr
```

### FSOP
> 通过覆盖 IO_list_all, 使其指向伪造的 \_IO_FILE_plus, 并且该\_IO_FILE_plus中的指针vtable指向伪造的vtable
> 然后通过close()、exit(0)、abort，使程序或者文件关闭,程序或者文件关闭时,需要刷新缓存,此时会调用vtable中的__overflow
> 再基于 fake vtable 的攻击就可以将 调用__overflow() --> system('/bin/sh\x00')

#### 攻击前提
> leak libc
> 任意写
> 在伪造 IO_FILE_plus时:
		+ fp->\_mode <= 0
		+ fp->\_IO_write_ptr > fp->\_IO_write_base

#### 利用流程
> fake IO_FILE_plus, fkae vtable(change {vtable->\_\_overflow} --> system)
> 覆盖_IO_list_all,使其指向伪造的 IO_FILE_plus
> 程序执行完成,exit,abort

### \_IO_FILE结构体攻击: make FILE structure great again
> \_IO_FILE结构体中, 有_IO_buf_base这些指针
> 改变这些指针以达到任意读任意写的目的

#### 任意写
![arbitrary_write](https://www.github.com/Byzero512/blog_img/raw/master/1538660920987.png)

#### 任意读
![arbitrary_read](https://www.github.com/Byzero512/blog_img/raw/master/1538660996822.png)


### 总结
> 1. 改vtable中的函数指针从 glibc 2.23 开始不能用了, 于是转向了伪造整个vtable
> 2. 伪造整个vtable从 glibc 2.24 开始不能用了, 于是转向了 \_IO_FILE_plus 中的 IO_FILE 结构体, 控制里面的一些读写指针