---
title: CentOS7下安装shadowsocks和proxychains4
date: 2018-06-08 14:57:30
categories:
  - Linux
tags:
  - centos7
  - shadowsocks
  - proxychains
---

国内服务器使用composer时候经常会出现这样那样的问题，一般我的解决办法是海外买一部vps作为学习机，国内服务器通过学习机...你懂的。

<!--more-->

## 安装shadowsocks

安装所需依赖，下载源码，编译安装。

```bash
yum install epel-release -y
yum install gcc gettext autoconf libtool automake make pcre-devel asciidoc xmlto udns-devel libev-devel mbedtls-devel libsodium-devel c-ares-devel -y

cd /usr/local/src
git clone https://github.com/shadowsocks/shadowsocks-libev.git
cd shadowsocks-libev
git submodule update --init --recursive

./autogen.sh
./configure
make && make install
```

### 作为服务端配置

创建配置文件，

```bash
mkdir -p /usr/local/etc/shadowsocks
vi /usr/local/etc/shadowsocks/server.json
```

文件内容，

```
{
    "server": "0.0.0.0",
    "server_port": 30023,
    "password": "",
    "method": "chacha20"
}
```

添加服务管理文件/etc/systemd/system/ss-server.service，并跟随开机启动。

```
[Unit]
Description=Shadowsocks Server

[Service]
TimeoutStartSec=0
ExecStart=/usr/local/bin/ss-server -c /usr/local/etc/shadowsocks/server.json

[Install]
WantedBy=multi-user.target
```

```bash
systemctl enable ss-server
systemctl start ss-server
```

### 作为客户端配置

创建配置文件，

```bash
mkdir -p /usr/local/etc/shadowsocks
vi /usr/local/etc/shadowsocks/client.json
```

文件内容，

```
{
    "server": "0.0.0.0",
    "server_port": 10000,
    "local_address": "127.0.0.1",
    "local_port": 1086,
    "password": "yourpasswd",
    "timeout": 300,
    "method": "aes-192-cfb",
    "workers": 1
}
```

添加服务管理文件/etc/systemd/system/ss-client.service，并跟随开机启动。

```
[Unit]
Description=Shadowsocks Client

[Service]
TimeoutStartSec=0
ExecStart=/usr/local/bin/ss-local -c /usr/local/etc/shadowsocks/client.json

[Install]
WantedBy=multi-user.target
```

```bash
systemctl enable ss-client.service
systemctl start ss-client.service
```

## proxychains4

安装，

```bash
cd /usr/local/src
git clone https://github.com/rofl0r/proxychains-ng.git
cd proxychains-ng
./configure
make && make install
make install-config
```

修改配置文件/usr/local/etc/proxychains.conf，

```
...
chain_len = 1
...
localnet 127.0.0.0/255.0.0.0 
localnet 192.168.0.0/255.255.0.0
...
[ProxyList]
socks5 127.0.0.1 1086
```

完，之后使用composer等命令行工具的时候，使用下面命令即可。

```bash
proxychains4 composer update
```

