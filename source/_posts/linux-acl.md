---
title: Linux ACL权限设置
date: 2016-11-17 20:56:26
tags:
	- linux
	- acl
---
这篇文章是讨论关于setfacl和getfacl这个命令的。

<!--more-->

## setfacl选项

先看看这个命令的参数，

> -b,--remove-all：删除所有扩展的acl规则，基本的acl规则(所有者，群组，其他）将被保留。
> -k,--remove-default：删除缺省的acl规则。如果没有缺省规则，将不提示。
> -n，--no-mask：不要重新计算有效权限。setfacl默认会重新计算ACL mask，除非mask被明确的制定。
> --mask：重新计算有效权限，即使ACL mask被明确指定。
> -d，--default：设定默认的acl规则。
> --restore=file：从文件恢复备份的acl规则（这些文件可由getfacl -R产生）。通过这种机制可以恢复整个目录树的acl规则。此参数不能和除--test以外的任何参数一同执行。
> --test：测试模式，不会改变任何文件的acl规则，操作后的acl规格将被列出。
> -R，--recursive：递归的对所有文件及目录进行操作。
> -L，--logical：跟踪符号链接，默认情况下只跟踪符号链接文件，跳过符号链接目录。
> -P，--physical：跳过所有符号链接，包括符号链接文件。 
> --version：输出setfacl的版本号并退出。
> --help：输出帮助信息。 
> --：标识命令行参数结束，其后的所有参数都将被认为是文件名 
> -：如果文件名是-，则setfacl将从标准输入读取文件名。

我们不讨论太多理论的东西，直接进入实际应用部分。

## 应用

场景：我负责公司一个web测试服务器的运维，拥有root账号，服务器上运行着例如nginx(www)、mysql(mysql)、php等等服务。好了，有一天老板要求他开一个叫做webuser的账号，是/data/www（这个目录用来存放一些web项目的源码）目录的owner，平时开发的同事可以使用这个账号登陆，做一些代码更新的动作。

> 按照习惯，root账号操作我们用#开头，webuser账号操作我们用$开头。

首先创建目录，并把目录的所有者授权给webuser。
```bash
# mkdir -p /data/www
# chown webuser:webuser /data/www
```

然后设置下acl权限，让www用户可以访问到这个目录，
```bash
$ setfacl -d -m user:www:x /data/www
```

这时我们查看下acl定义后的权限，
```bash
$ getfacl /data/www
# file: www
# owner: webuser
# group: webuser
user::rwx
user:www:--x
group::---
mask::--x
other::---
```

前面三个#定义了文件的名称以及file owner和group等信息。
剩下的5行分别我们称之为Access Entry，每一条Access Entry定义了特定的类别可以对文件拥有的操作权限。它由三部分组成。

> 标记类型:限定符(可选):权限位

其中标记类型包括以下几种，

> ACL_USER_OBJ：相当于Linux里file_owner的permission
> ACL_USER：定义了额外的用户可以对此文件拥有的permission 
> ACL_GROUP_OBJ：相当于Linux里group的permission 
> ACL_GROUP：定义了额外的组可以对此文件拥有的permission 
> ACL_MASK：定义了ACL_USER, ACL_GROUP_OBJ和ACL_GROUP的最大权限
> ACL_OTHER：相当于Linux里other的permission

当然有人问为什么getfacl只出现了五条，因为我只设定了额外用户，并没有设定额外的组...

## ACL_MASK讨论

我们再过头来看下ACL_MASK这个标记类型，它很重要。在Linux中大家都知道这样一组rw-rw-r--，第4到6位代表的组权限，但是这只是在ACL_MASK不存在的情况下成立，如果文件有ACL_MASK（ls -al后面有个加号）值，那么它代表的就是mask值而不是组权限了。

我们做个测试，比如我们有个脚本test.sh，

```bash
$ ls -l test.sh
-rwxrw-r-- 1 root admin 0 Jul  7 10:34 test.sh
```
这时候它的组权限是rw，没有执行权限，

```bash
$ setfacl -m user:webuser:rwx ./test.sh
$ getfacl --omit-header ./test.sh
user::rwx
user:webuser:rwx
group::rw-
mask::rwx
other::r--

$ ls -l test.sh
-rwxrwxr--+ 1 root admin 0 Jul  7 10:34 test.sh
```

我们可以看到这时候组权限已经变成了rwx，但是如果我们用admin组的用户执行test.sh，其实是会permission deny的，这里要注意下。

我们继续研究mask，我们现在把test.sh的mask改为r--，那么admin组还能有写权限吗？

```bash
$ setfacl -m mask::r-- ./test.sh
$ getfacl --omit-header ./test.sh
user::rwx
user:webuser:rwx  #effective:r--
group::rw-        #effective:r--
mask::r--
other::r--
```

我们可以注意到，ACL_USER和ACL_GROUP_OBJ旁边多了一个#effective:r--，我们回顾下，ACL_MASK的定义：它规定了ACL_USER，ACL_GROUP_OBJ和ACL_GROUP的最大权限。那么在我们这个例子中它们的最大权限也就是read，所以即使我们给ACL_USER和ACL_GROUP_OBJ设置了其它权限，但是他们真正有效果的只有read权限。

## ACL默认值

大家都注意我之前对/data/www目录设置ACL的时候加了一个-d参数，对于网站而言，我们时时刻刻会有新文件（目录也是文件）产生，如果需要每个新建文件都去设置，是不实际的（事实上有些目录是运行中的程序产生的），还是再来个例子。

```bash
$ mkdir dir
```

我们希望dir目录下的所有文件www都能访问，

```bash
$ setfacl -d -m user:www:rw ./dir
$ getfacl --omit-header ./dir
user::rwx
group::r-x
other::r-x
default:user::rwx
default:user:www:rw-
default:group::r-x
default:mask::rwx
default:other::r-x
```

这里我们看到ACL信息中多了一个default项，www用户默认拥有rwx权限，现在我们在dir下面新建一个test.txt文件，

```bash
$ touch ./dir/test.txt
$ ls -l ./dir/test.txt
-rw-rw-r--+ webuser webuser 0 Nov 17 22:13 test.txt

$ getfacl --omit-header ./dir/test.txt
user::rw-
user:www:rw-
group::r-x #effective:r--
mask::rw-
other::r--
```

这里我们可以看到dir下建立的文件，www用户自动有用了read,write权限。

基本资料就这些了，剩下的查阅手册看看这两个命令的用法就可以搞懂了。