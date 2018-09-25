---
title: python中的struct模块
description: 本文主要记录了python的用来pack与unpack的模块: struct
author: byzero(or named Byzero512)
categories:
- python
tags: 
- python
---

### struct模块
> 此模块可以实现字符串与数据类型的转换,如对浮点数的一些处理
> 感觉这个模块有点像格式化字符串函数
> pack后的数据就是放在内存里的样子

```python
struct.pack(fmt,v1,v2); v1,v2是特定的数字
struct.unpack(fmt,string)

```

#### fmt
1. endian

![endian](https://www.github.com/Byzero512/blog_img/raw/master/1537881586739.png)

2. dataType

![dataType](https://www.github.com/Byzero512/blog_img/raw/master/1537881550678.png)



### 浮点数

#### 已知IEEE标准下的浮点数,求10进制科学计数法下的浮点数
```python
print("%.20e"%struct.unpack("<d",p64(----Num-----))[0])
# 也可以以float类型unpack,pack

# Example:
>>> print("%.20e"%struct.unpack("<d",p64(0xffff))[0])
3.23785921002060922726e-319
```

#### 已知10进制科学计数法下的浮点数,求IEEE标准下的浮点数
```python
print(struct.pack("<d",dec_Num).encode("hex"))

# Example:
>>> print(struct.pack(">d",3.23785921002060922726e-319).encode("hex"))
000000000000ffff

```