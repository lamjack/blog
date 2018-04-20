---
title: CentOS7 Python 开发环境搭建
date: 2016-11-09 10:38:45
categories:
  - 编程语言
  - Python
tags:
  - centos7
  - python
---
CentOS7系统默认使用的Python版本为2.7.5。

## 准备工作

### 更新系统组件

```bash
yum update -y
```

### 安装常用包

安装一些常用的工具集，比如gcc，

```bash
yum groupinstall -y development
```

## 安装pip、virtualenv

### pip

```bash
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
```

### virtualenv

```bash
pip install virtualenv
```

## 安装Supervisor

首先通过pip安装supervisor，

```bash
pip install supervisor
```

添加配置文件以及程序目录，

```bash
echo_supervisord_conf > /etc/supervisord.conf
mkdir -p /etc/supervisord.d/
```

编辑配置文件/etc/supervisord.conf，加入

```ini
[include]
files = /etc/supervisord.d/*.conf
```

此处的/etc/supervisord.d/用于存放各种程序的supervisord启动脚本（后缀为conf）。

然后添加supervisor的systemd启动service控制命令，

编辑/usr/lib/systemd/system/supervisord.service，文件内容：

```ini
[Unit]
Description=Supervisord

[Service]
Type=forking
PIDFile=/tmp/supervisord.pid
ExecStart=/usr/bin/supervisord -c /etc/supervisord.conf
ExecStop=/bin/kill -TERM $MAINPID
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

配置开机启动并启动supervisord服务，

```bash
systemctl enable supervisord
systemctl start supervisord
```
