---
title: Redis出现Overcommit memory问题分析及解决
date: 2017-03-02 10:56:35
tags:
    - redis
    - memory
excerpt: 122
---

## 问题描述

今天公司一个项目出现了间歇性用户无法登陆，经测试初步排除了代码问题，估计是保存登陆 Session 的服务（Redis）出现问题，但是 telnel 6379 端口正常，尝试进行读写操作，发现提示 not permitted 错误。

<!--more-->

> WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.

## 解决方案

调整 Redis 最大占用内存，编辑配置文件，

```conf
# 最大占用内存4G
maxmemory 4294967296
# 优先删除不常使用的key
maxmemory-policy allkeys-lru
# LRU样本数量
maxmemory-samples 5
```

重启 Redis 服务。

修改内核参数，编辑 /etc/sysctl.conf 文件，

```conf
...
vm.overcommit_memory = 1
...
```

使配置生效，

```bash
sysctl -p
```

## 问题总结

### Linux 的 Memory Overcommit

overcommit_memory 有三个可用值，（由于没有找到好的翻译，怕产生错误或歧义，我这里贴上官方的中英文）。

> 0 - 默认设置。内核执行启发式内存过量使用处理，方法是估算可用内存量，并拒绝明显无效的请求。遗憾的是因为内存是使用启发式而非准确算法计算进行部署，这个设置有时可能会造成系统中的可用内存超载。
> 1 - 内核执行无内存过量使用处理。使用这个设置会增大内存超载的可能性，但也可以增强大量使用内存任务的性能。
> 2 - 内存拒绝等于或者大于总可用 swap 大小以及 overcommit_ratio 指定的物理 RAM 比例的内存请求。如果您希望减小内存过度使用的风险，这个设置就是最好的。

这是英文版，

> 0 - Heuristic overcommit handling. Obvious overcommits of address space are refused. Used for a typical system. It ensures a seriously wild allocation fails while allowing overcommit to reduce swap usage. root is allowed to allocate slightly more memory in this mode. This is the default.
> 1 - Always overcommit. Appropriate for some scientific applications. Classic example is code using sparse arrays and just relying on the virtual memory consisting almost entirely of zero pages.
> 2 - Don't overcommit. The total address space commit for the system is not permitted to exceed swap + a configurable amount (default is 50%) of physical RAM. Depending on the amount you use, in most situations this means a process will not be killed while accessing pages but will receive errors on memory allocation as appropriate.
> Useful for applications that want to guarantee their memory allocations will be available in the future without having to initialize every page.

这里用银行提款的例子来解释下，

银行有现金 1000，甲乙丙三人需要取款，银行要求取款人必须先提前预约取款才能提现。第一种情况，甲预约要提 1000，银行说可以；乙说要提 800，银行说可以；丙说要提 1200，银行拒绝；第二种情况，提多少都可以，没钱了银行就关门；第三种情况，银行计算定一个总额，超过就拒绝；

### Redis LRU

LRU(Least Recently Used)**最近\*\***最少\*\*使用算法是众多置换算法中的一种。

具体参考下[这篇文章](http://blog.csdn.net/cjfeii/article/details/47259519)，这里只记录下置换策略的选项。

> noeviction: 不进行置换，表示即使内存达到上限也不进行置换，所有能引起内存增加的命令都会返回 error
> allkeys-lru: 优先删除掉最近最不经常使用的 key，用以保存新数据
> volatile-lru: 只从设置失效（expire set）的 key 中选择最近最不经常使用的 key 进行删除，用以保存新数据
> allkeys-random: 随机从 all-keys 中选择一些 key 进行删除，用以保存新数据
> volatile-random: 只从设置失效（expire set）的 key 中，选择一些 key 进行删除，用以保存新数据
> volatile-ttl: 只从设置失效（expire set）的 key 中，选出存活时间（TTL）最短的 key 进行删除，用以保存新数据

参考文章：
[理解 LINUX 的 MEMORY OVERCOMMIT](http://linuxperf.com/?p=102)
[如何用 Redis 做 LRU-Cache](http://blog.csdn.net/cjfeii/article/details/47259519)
[RHEL 文档 - 5.4 容量调节](https://access.redhat.com/documentation/zh-CN/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-memory-captun.html)
[redis overcommit memory (oom) 问题解决](http://blog.51yip.com/nosql/1776.html)
