---
title: CentOS 7 firewalld 研习记录
date: 2016-11-08 12:35:28
categories:
  - Linux
tags: 
  - centos7
  - firewalld
---
<!--more-->

## firewalld与iptables比较

1. firewalld可以动态修改单挑规则，而不需要像iptables那样，在修改了规则后必须全部刷新才生效；
2. firewalld在使用上要比iptables人性化很多，即使不明白“五张表五条链”，而且对TCP/IP不理解也可以实现大部分功能；

firewalld自身并不具备防火墙的功能，而是和iptables一样需要通过内核的netfilter来实现，也就是说firewalld和iptables一样，他们的作用都是用于维护规则，而真正使用规则干活的是内核的netfilter，只不过firewalld和iptables的结构以及使用方法不一样罢了。

![firewall_stack](https://jack-images.wilead.net/blog/q5w6u.png)

## firewalld安装以及使用
在 Red Hat Enterprise Linux 7 中，默认安装 firewalld 和图形化用户接口配置工具 firewall-config。作为 root 用户运行下列命令可以检查：

```bash
yum install firewalld firewall-config
```

### 安装防火墙
要安装firewalld，则以 root 用户身份运行以下命令：

```bash
yum install firewalld
```

### 启动防火墙
要启动 firewalld，则以 root 用户身份输入以下命令：

```bash
systemctl enable firewalld
systemctl start firewalld
```

### 禁用防火墙
要禁用 firewalld，则作为 root 用户运行下列命令：

```bash
systemctl disable firewalld
systemctl stop firewalld
```

### 检查防火墙是否运行
如果 firewalld 在运行，输入以下命令检查：

```bash
systemctl status firewalld
```

另外，检查 firewall-cmd 是否可以通过输入以下命令来连接后台程序：

```bash
firewall-cmd --state
```

### 检查当前防火墙版本
命令行工具 firewall-cmd 是默认安装的应用程序 firewalld 的一部分。您可以查证到它是为检查版本或者展示帮助结果而安装的。输入如下命令来检查版本：

```bash
firewall-cmd --version
```

## firewalld的配置
### 配置文件模式
firewalld的配置文件以xml格式为主（主配置文件firewalld.conf例外），他们有两个存储位置，

1. /etc/firewalld/
2. /usr/lib/firewalld/

使用时的规则是这样的：当需要一个文件时firewalld会首先到第一个目录中去查找，如果可以找到，那么就直接使用，否则会继续到第二个目录中查找。

firewalld的这种配置文件结构的主要作用是这样的：在第二个目录中存放的是firewalld给提供的通用配置文件，如果我们想修改配置，那么可以copy一份到第一个目录中，然后再进行修改。这么做有两个好处：首先我们日后可以非常清晰地看到都有哪些文件是我们自己创建或者修改过的，其次，如果想恢复firewalld给提供的默认配置，只需要将自己在第一个目录中的配置文件删除即可，非常简单，而不需要像其他很多软件那样在修改之前还得先备份一下，而且时间长了还有可能忘掉之前备份的是什么版本。

### 配置文件结构
firewalld的配置文件结构非常简单，主要有两个文件和三个目录：

文件：firewalld.conf、lockdown-whitelist.xml

目录：zones、services、icmptypes

另外，如果使用到direct，还会有一个direct.xml文件。我们要注意，在保存默认配置的目录/usr/lib/firewalld/中只有我们这里所说的目录，而没有firewalld.conf、lockdown-whitelist.xml和direct.xml这三个文件，也就是说这三个文件只存在于“/etc/firewalld/”目录中。

#### firewalld.conf

firewalld的主配置文件，是键值对的格式，不过非常简单，只有五个配置项。

* DefaultZone：默认使用的zone，默认值为public；
* MinimalMark：linux内核会对每个进入的数据包都进行标记，这里用于设置标记的最小值；
* CleanupOnExit：这个配置项非常容易理解，它表示当退出firewalld后是否清除防火墙规则，默认值为yes；
* Lockdown：这个选项跟D-BUS接口操作firewalld有关，firewalld可以让别的程序通过D-BUS接口直接操作，当Lockdown设置为yes的 时候就可以通过lockdown-whitelist.xml文件来限制都有哪些程序可以对其进行操作，而当设置为no的时候就没有限制了，默认值为no；
* IPv6_rpfilter：其功能类似于rp_filter，只不过是针对ipv6版的，其作用是判断所接受到的包是否是伪造的，检查方式主要是通过路由表中的路由条目实现的，更多详细的信息大家可以搜索uRPF相关的资料，这里的默认值为yes；

#### lockdown-whitelist.xml

当Lockdown为yes的时候用来限制可以通过D-BUS接口操作firewalld的程序。

#### direct.xml

通过这个文件可以直接使用防火墙的过滤规则，这对于熟悉iptables的用户来说会非常顺手，另外也对从原来的iptables到firewalld的迁移提供了一条绿色通道。

#### zones

保存zone配置文件。

#### services

保存service配置文件。

#### icmptypes

保存和icmp类型相关的配置文件。

### zones网络区

firewalld默认提供了九个zone配置文件：block.xml、dmz.xml、drop.xml、external.xml、home.xml、internal.xml、public.xml、trusted.xml、work.xml，他们都保存在/usr/lib/firewalld/zones/目录下。这些zone之间是什么关系？他们分别适用用哪些场景呢？

官方对这九个区域是这样定义的，

**drop（丢弃）**
任何接收的网络数据包都被丢弃，没有任何回复。仅能有发送出去的网络连接。

**block（限制）**
任何接收的网络连接都被 IPv4 的 icmp-host-prohibited 信息和 IPv6 的 icmp6-adm-prohibited 信息所拒绝。

**public（公共）**
在公共区域内使用，不能相信网络内的其他计算机不会对您的计算机造成危害，只能接收经过选取的连接。

**external（外部）**
特别是为路由器启用了伪装功能的外部网。您不能信任来自网络的其他计算，不能相信它们不会对您的计算机造成危害，只能接收经过选择的连接。

**dmz（非军事区）**
用于您的非军事区内的电脑，此区域内可公开访问，可以有限地进入您的内部网络，仅仅接收经过选择的连接。

**work（工作）**
用于工作区。您可以基本相信网络内的其他电脑不会危害您的电脑。仅仅接收经过选择的连接。

**home（家庭）**
用于家庭网络。您可以基本信任网络内的其他计算机不会危害您的计算机。仅仅接收经过选择的连接。

九个zone其实就是九种方案，而且起决定作用的其实是每个xml文件所包含的内容，而不是文件名，所以大家不需要对每种zone（每个文件名）的含义花费过多的精力，比如trusted这个zone会信任所有的数据包，也就是说所有数据包都会放行，但是public这个zone只会放行其中所配置的服务，其他的一律不予放行，其实我们如果将这两个文件中的内容互换一下他们的规则就换过来了，也就是public这个zone会放行所有的数据包。下面我们来看一下这两个文件的内容：

public.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
</zone>
```

trusted.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<zone target="ACCEPT">
  <short>Trusted</short>
  <description>All network connections are accepted.</description>
</zone>
```

我们要特别注意trusted.xml中zone的target，就是因为他设置为了ACCEPT，所以才会放行所有的数据包，而public.xml中的zone没有target属性，这样就会默认拒绝通过，所以public这个zone（这种方案）只有其中配置过的服务才可以通过。

### service服务

服务用于将端口号改为服务名，这样有两个好处，

1. 使用服务名配置的语义清晰，不容易出错；
2. 在对某个服务的端口号进行修改的时候只需要修改相应的service文件就可以了，而不需要再修改zones；

这其实跟DNS将ip地址和域名关联了起来是一样的道理。

service配置文件的命名规则是<服务名>.xml，比如ssh的配置文件是ssh.xml，http的配置文件是http.xml等，他们默认保存在/usr/lib/firewalld/services/目录下，常见的服务其中都可以找到，如果我们想修改某个服务的配置，那么可以复制一份到/etc/firewalld/services/目录下然后进行修改就可以了，要想恢复默认配置直接将我们自己的配置文件删除就可以了。我们来看一下ssh服务的ssh.xml文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>SSH</short>
  <description>Secure Shell (SSH) is a protocol for logging into and executing commands on remote machines. It provides secure encrypted communications. If you plan on accessing your machine remotely via SSH over a firewalled interface, enable this option. You need the openssh-server package installed for this option to be useful.</description>
  <port protocol="tcp" port="22"/>
</service>
```

可以看到这里配置了tcp的22号端口，所以将ssh服务配置到所使用的zone（默认public）中后tcp的22号端口就开放了。如果我们想将ssh的端口修改为222，那么只需要将ssh.xml复制一份到/firewalld/services/中，然后将端口号修改为222就可以了。当然直接修改/usr/lib/firewalld/services/中的配置文件也可以实现，但是强烈建议不要那么做，原因相信大家都明白。

明白原理之后使用起来就可以非常灵活了，比如我们将/etc/firewalld/services/ssh.xml文件复制一份到“/etc/firewalld/services/中，然后将名字改为abc.xml，并且将abc这个服务配置到所使用的zone中，这时22端口就会开放。也就是说在zone中所配置的服务其实跟实际的服务并不存在直接联系，而是和相应配置文件中配置的内容有关系。

## 常用命令以及常见问题

如果命令加--permanent代表永久的意思，否则reload之后会消失。

### 显示目前的设定

```bash
firewall-cmd --list-all
```

### 永久开启、关闭某个服务

```bash
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --zone=public --remove-service=http --permanent

firewall-cmd --zone=public --add-port=8080/tcp --permanent 
firewall-cmd --zone=public --remove-port=8080/tcp --permanent
```
注意添加服务时候我没有加zone，则代表加到默认的zone里面。

### 限制某服务或某端口只允许指定IP连接

```bash
firewall-cmd --add-rich-rule="rule family="ipv4" source address="192.168.1.88" service name="ssh" "
firewall-cmd --add-rich-rule="rule family="ipv4" source address="192.168.1.0/24" service name="ssh" accept"

firewall-cmd --add-rich-rule="rule family="ipv4" source address="192.168.12.9" port port="8080" protocol="tcp" accept"
```

### 端口转发
将本机某端口的数据包转发到另外一个端口

```bash
firewall-cmd --zone="public" --add-forward-port=port=80:proto=tcp:toport=8080
```

将本机某端口的数据包转发到另外一台主机的端口

```bash
firewall-cmd --zone="public" --add-forward-port=port=80:proto=tcp:toport=8080:toaddr=x.x.x.x
```

重载配置文件

```bash
firewall-cmd reload
```

## 参考文章
+ [Firewalld的结构](http://www.excelib.com/article/287/show/)
+ [红帽企业版Linux7-安全性指南-使用防火墙](https://my.oschina.net/Jacker/blog/32936)