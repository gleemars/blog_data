---
title: gdbinit与python脚本
description: 本文主要记录如何使用python拓展gdb
author: byzero(or named Byzero512)
categories:
- python
- gdb
tags:
- python
- gdb
---

### 定义函数

``` python
     class Greet (gdb.Function):
     
       def __init__ (self):
         super (Greet, self).__init__ ("greet")
     
       def invoke (self, name):
         return "Hello, %s!" % name.string ()
     
     Greet ()
```
### 定义命令

``` python
class HelloWorld (gdb.Command):
  """Greet the whole world."""

  def __init__ (self):
    super (HelloWorld, self).__init__ ("hello-world", gdb.COMMAND_USER)

  def invoke (self, arg, from_tty):
    print "Hello, World!"

HelloWorld ()
```

### api接口

``` python
import gdb

string=gdb.execute(command,to_string=True) 
//可以将gdb执行的命令,并以字符串的方式返回

list=gdb.string_to_argv(args)
//可以将命令的参数由字符串转换成一个列表

gdb.parse_and_eval (expression)
```