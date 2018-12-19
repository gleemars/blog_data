---
title: java 异常 文件流
tags: 
categorires: 
author: 
top: 
---

## 异常
> 必检异常需要强制要求使用try-catch块进行处理, 例如 io 异常
> 必检异常在使用时需要先声明

### 异常声明
> 函数原型后面添加 异常声明

```java
public static func() throws exception1,exception2{
	// body
}

public static funcname()
	throws exception1{
	try{
		throws new exception1;
	}
	catch (exception1 | exception2 ex){
		//body
	}

	finally{
	// body
	}
}

```

### 自定义异常
> 需要先注册
> 异常抛出现在函数内部处理

```java
public class exceptdiy extends Exception{

	public exceptdiy() {
		System.out.println("exception diy");
	}
}
```


## 文件读写

### Scanner 和 PrintWriter
```java
import java.io.File
import java.util.Scanner
import java.io.PrintWriter
String fpath="file path";
File fileobj=new File()


```