---
title: 查找正在监听端口的进程
date: 2018-06-07 12:39:47
categories:
  - Linux
tags:
  - process
---

这篇文章将介绍几个命令，帮助你找出每个开放端口对应的相关进程，首先了解下下面几个命令或文件系统。

**netstat** - 可用于列出系统上所有的网络套接字连接情况，包括 tcp, udp 以及 unix 套接字，另外它还能列出处于监听状态（即等待接入请求）的套接字。

**fuser** - 使用文件或文件结构识别进程。

**lsof** - 查看当前系统文件的工具。在Linux环境下，任何事物都以文件的形式存在，通过文件不仅仅可以访问常规数据，还可以访问网络连接和硬件。如传输控制协议 (TCP) 和用户数据报协议 (UDP) 套接字等，系统在后台都为该应用程序分配了一个文件描述符，该文件描述符提供了大量关于这个应用程序本身的信息。

**/proc/$pid** - /proc目录是一种文件系统，即proc文件系统。与其它常见的文件系统不同的是，/proc是一种伪文件系统（也即虚拟文件系统），存储的是当前内核运行状态的一系列特殊文件，用户可以通过这些文件查看有关系统硬件及当前正在运行进程的信息，甚至可以通过更改其中某些文件来改变内核的运行状态。 

<!--more-->

以下命令必须以root用户身份运行。

## netstat

```bash
netstat -tulpn
```

![1](https://jack-images.wilead.net/blog/tcoja.png)

可以看到mysqld进程25485正在监听3306端口，我们可以通过/proc文件系统验证下。

```bash
ls -l /proc/25485/exe
```

![2](https://jack-images.wilead.net/blog/vrdht.png)

还可以通过grep命令来过滤查询信息。

```bash
netstat -tulpn | grep :80
```

![3](https://jack-images.wilead.net/blog/6we8v.png)

## fuser

我们通过fuser命令来查看当前本机的6379端口被哪个进程监听，

```bash
fuser 6379/tcp
```

![4](https://jack-images.wilead.net/blog/w49zo.png)

通过/proc文件查询下PID25382对应的进程信息，

```bash
ls -l /proc/25382/exe
```

![5](https://jack-images.wilead.net/blog/jt4w4.png)

## 实战1：查询某个进程的工作目录（Working Directory）

```bash
ls -l /proc/25485/cwd
```

![1](https://jack-images.wilead.net/blog/4uyek.png)

这跟pwdx命令的功能是相同的。

## 实战2：找出进程的所有者（Owner）

```bash
ps aux | grep 25485
```

![屏幕快照 2018-06-07 下午2.45.07](https://jack-images.wilead.net/blog/rhc83.png)

或者通过/proc查询，

```bash
grep --color -w -a USER /proc/25485/environ
```

![2](https://jack-images.wilead.net/blog/uqzbg.png)

## lsof

列举下lsof命令的几种常用的查询方式，

```bash
lsof -i :port
lsof -i tcp:port
lsof -i udp:port
lsof -i :port | grep LISTEN
```

![3](https://jack-images.wilead.net/blog/ywgkg.png)

查询更多关于nginx主进程1837的信息，

```bash
ps aux | grep 1837
```

![4](https://jack-images.wilead.net/blog/c7a4d.png)

指定要查询的信息，

```bash
ps -eo pid,user,group,args,etime,lstart | grep '[1]837'
```

![5](https://jack-images.wilead.net/blog/rgtqu.png)

其中包含的信息有，

> 进程ID - 1837
>
> 所有者名称 - root
>
> 所有组名称 - root
>
> 进程执行参数 - /usr/sbin/nginx -c /etc/nginx/nginx.conf
>
> 进程开始到现在的时间 - 3-03:53:20，格式为[[dd-]hh:]mm:ss
>
> 最后启动的时间 - Mon Jun 4 10:59:49 2018



参考资料：

[Linux: Find Out Which Process Is Listening Upon a Port](https://www.cyberciti.biz/faq/what-process-has-open-linux-port/)

[深入理解linux系统下proc文件系统内容](https://www.cnblogs.com/cute/archive/2011/04/20/2022280.html)

[lsof 一切皆文件](http://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/lsof.html)