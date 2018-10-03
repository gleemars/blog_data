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
  int _flags;       /* High-order word is _IO_MAGIC; rest is flags. */
#define _IO_file_flags _flags

  /* The following pointers correspond to the C++ streambuf protocol. */
  /* Note:  Tk uses the _IO_read_ptr and _IO_read_end fields directly. */
  char* _IO_read_ptr;   /* Current read pointer */
  char* _IO_read_end;   /* End of get area. */
  char* _IO_read_base;  /* Start of putback+get area. */
  char* _IO_write_base; /* Start of put area. */
  char* _IO_write_ptr;  /* Current put pointer. */
  char* _IO_write_end;  /* End of put area. */
  char* _IO_buf_base;   /* Start of reserve area. */
  char* _IO_buf_end;    /* End of reserve area. */
  
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
void * funcs[] = {
   1 NULL, // "extra word"
   2 NULL, // DUMMY
   3 exit, // finish
   4 NULL, // overflow
   5 NULL, // underflow
   6 NULL, // uflow
   7 NULL, // pbackfail
   
   8 NULL, // xsputn  #printf
   9 NULL, // xsgetn
   10 NULL, // seekoff
   11 NULL, // seekpos
   12 NULL, // setbuf
   13 NULL, // sync
   14 NULL, // doallocate
   15 NULL, // read
   16 NULL, // write
   17 NULL, // seek
   18 pwn,  // close
   19 NULL, // stat
   20 NULL, // showmanyc
   21 NULL, // imbue
};
```

3.  \_IO_FILE_plus
> 封装了上面两个结构

```cpp
struct _IO_FILE_plus
{
	_IO_FILE file;
	IO_jump_t * vtable;
};
```

## 文件IO操作
> 从IO库函数 --> FILE文件结构 --> 文件结构vtable中对应的函数 --> 系统调用

### 打开与关闭文件
#### fopen()
> FILE *fopen(char *filename, *type);

1. 在堆中创建FILE结构, 并初始化vtable
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

![printf/puts](https://www.github.com/Byzero512/blog_img/raw/master/1538553969841.png) 


