---
title: MySQL5 主从复制以及主主双机热备份设置
date: 2018-07-13 11:55:17
categories:
  - 数据库
  - MySQL
tags: 
  - mysql
  - replication
---

配置之前，如果是线上环境，我们需要对数据库进行数据同步的工作，这时候就涉及到数据库锁。

```mysql
FLUSH TABLES WITH READ LOCK;
UNLOCK TABLES;
```

上面这个命令是**全局读锁定**，执行命令后所有库的表都被锁定只读，这时候数据库的写操作将被阻塞，只可以进行读操作。MySQL 包括以下两种锁，

> READ LOCK: 允许其它并发的读请求，但阻塞写请求，即可以同时读，但不允许任何写，也叫共享锁；
>
> WRITE LOCK: 不允许其它并发的读和写请求，是排它的(exclusive)，也叫独占锁。

补充下表锁的命令，

```mysql
LOCK TABLES table_name [AS alias] {READ [LOCAL] | [LOW_PRIORITY] WRITE}
UNLOCK TABLES;
```

另外需要了解的是，**全局锁**和**表锁**都有隐式提交语句，在退出 MySQL 终端的时候执行 UNLOCK TABLES，也就是说如果需要一直生效就要保持会话(session)。

下面让我们开始进行配置。

<!--more-->

## MySQL配置

修改 my.cnf 文件，下面是 master 节点的配置，

```
[mysqld]
log-bin = mysql-bin
server-id = xxx # 注意不同 MySQL 实例的 server-id 不能相同

binlog-ignore-db = mysql,information_schema,performance_schema # 忽略的DB

# master-master 的情况下，主键增量要不同
auto-increment-offset = 1
auto-increment-increment = 1
```

下面是 slave 节点的配置，

```
[mysqld]
log-bin = mysql-bin
server-id = xxx

## 开启中继日志，复制线程先把远程的变化复制到中继日志中，再执行
relay_log = relay-bin
log-slave-updates = ON

## 跳过所有的sql语句失败
slave-skip-errors = all
```

如果只需要部分数据库的改动写进二进制文件，使用 binlog-do-db，多个数据库用逗号分隔。

同样，如果作为slave，只需要同步对应数据库，使用 replicate-do-db，多个数据库逗号分隔。

作为master或者slave如果需要忽略某些db，分别使用 binlog-ignore-db 和 replicate-ignore-db 选项，多个用逗号分隔。

在 master 节点创建一个用于主从同步的用户，

```mysql
GRANT REPLICATION SLAVE ON *.* to '用户名'@'%' IDENTIFIED BY '密码'
```

现在查看下 master 节点的状态，

```mysql
SHOW MASTER STATUS;
```

我们可以看到下面的数据，他们分别代表当前二进制文件的名称以及开始语句的位置。

|       FILE       | Position |
| :--------------: | :------: |
| mysql-bin.000001 |   1000   |

进入 slave 节点，配置 master 信息，

```mysql
HANGE MASTER TO MASTER_HOST='ip', MASTER_USER='用户名', MASTER_PASSWORD='密码', MASTER_LOG_FILE='日志文件名', MASTER_LOG_POS=当前位置
```

启动 slave，并查看状态，

```mysql
START SLAVE;
SHOW SLAVE STATUS;
```

如果 Slave_IO_Running 和 Slave_SQL_Running 都为 Yes，则代表开始同步。

**如果出现错误**，<u>确保数据库已经锁定</u>的情况下，再查看一次 master status，然后设置 slave 即可。

## 主主双机热备份

与 master-slave 类似，把 master 当做 slave，把 slave 当做 master 再配置一次即可...

## 多线程主从复制

数据库复制的主要性能问题就是**数据延时**，为了优化性能，MySQL 5.6 引入了"多线程复制"这个新功能。

但 5.6 中的每个线程只能处理一个数据库，所以如果只有一个数据库，或者绝大多数写操作都是集中在某一个数据库的，那么这个“多线程复制”就不能充分发挥作用了。

MySQL 5.7 对 “多线程复制” 进行了改善，可以按照**逻辑时钟**的方式来分配线程，大大提高了复制性能。

首先查看当前 slave 节点的进程状态，

```mysql
SHOW PROCESSLIST;
```

![mysql-show-processlist](https://jack-images.wilead.net/blog/rq0as.png)

只有一个复制线程在运行。

修改 slave 节点的并发类型，并设置并发数量，

```mysql
SET GLOBAL SLAVE_PARALLEL_TYPE = 'logical_clock';
SET GLOBAL SLAVE_PARALLEL_WORKERS = 4;
```

![update_slave_parallel_type](https://jack-images.wilead.net/blog/ud0hh.png)

注意，使用会话设置的参数会在会话结束的时候失效，这里只是为了演示，配置参数可以写在 my.cnf 中。

```
[mysqld]
slave_parallel_type = 'logical_clock'
slave_parallel_workers = 4
```

下面我们再查看下 slave 的进程状态，会发现多了四个线程。

![show_processlist2](https://jack-images.wilead.net/blog/yp6lh.png)

完。