---
title: pwn for beginner
description: 2018, kap0k to beginner 
---


### 前置
1. python,编程语言很重要
> http://www.runoob.com/python/python-tutorial.html

### 环境
> 环境安装可能麻烦一些,请耐心一些,有问题在群里面问.

1.环境搭建视频
> https://www.bilibili.com/video/av14550200?from=search&seid=11015206578758826690

2. linux(建议使用ubuntu16.04,可以使用vmware安装虚拟机)
> 熟悉一些linux基本命令

3. ida pro 7.0 (必备神器)
> 版权原因,谷歌搜一搜

4. pwngdb(调试用)
> https://github.com/scwuaptx/Pwngdb

5. ROPgadget
> https://github.com/JonathanSalwan/ROPgadget

6. libcsearcher
> https://github.com/lieanu/LibcSearcher

7. pwntools(pwn题必备)
> https://github.com/giantbranch/pwn-env-init
> 这个网址是环境一键搭建脚本有
> 为64位系统提供32位运行环境支撑
下载了libc6的源码，方便源码调试
给gdb装上pwndbg和peda插件
安装pwntools

### 入门资料

#### 最全面的新手教程
> https://ctf-wiki.github.io/ctf-wiki/pwn/linux/stackoverflow/stack_intro/
> 对应题目放在
>  https://github.com/ctf-wiki/ctfchallenges/tree/master/pwn/stackoverflow

#### 前置

> 在我们 pwn程序之前需要突破程序的保护,前提是我们要了解保护机制
>http://yunnigu.dropsec.xyz/2016/10/08/checksec%E5%8F%8A%E5%85%B6%E5%8C%85%E5%90%AB%E7%9A%84%E4%BF%9D%E6%8A%A4%E6%9C%BA%E5%88%B6/

> 函数调用: https://segmentfault.com/a/1190000007977460

#### pwn 教程
> 招新考察范围: shellcode, rop, 格式化字符串....
> shellcode 的简单利用大家在 ctf-wiki 或者bamboofox 的视频里面可以学 

1. rop
> 来自bamboofox的视频教程
> https://www.youtube.com/watch?v=QYmWq2o7MtA&t=3047s
> 对应题目会发到群里(见群文件---> rop基础题目 )

2. 格式化字符串漏洞

> 可以看https://ctf-wiki.github.io/ctf-wiki/pwn/stackoverflow/stack_intro/的教程
> 也可以看以下视频:
> https://www.youtube.com/watch?v=FvGhDlK36PI&t=4s
https://www.youtube.com/watch?v=oiicCDA4RNc
https://www.youtube.com/watch?v=o3KXSayMvhw

3. plt表和got表(了解函数重定位过程)
> http://www.ifuryst.com/archives/Linux-PLT-GOT.html

### 其他推荐

1. angelboy的入门教程视频
> https://www.youtube.com/channel/UC_PU5Tk6AkDnhQgl5gARObA?app=desktop

2. Jarvis OJ(题目平台)
> https://www.jarvisoj.com/challenges

3. 一个适合新手的镜像
> https://exploit-exercises.com/protostar/
在Download中下载protostar的镜像然后安装
https://liveoverflow.com/binary_hacking/protostar/index.html

### tips
> 大一上学期会有一门课: 计算机概论, 大家认真听,认真学
> 计概不是水课哦


### 书籍推荐
> 王爽 <<汇编语言>>
> 程序员的自我修养
> 加密与解密 (目前只有第三版,第四版估计进度时间出)