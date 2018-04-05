---
title: Mac系统安装man命令中文文档
date: 2017-02-26 23:41:49
tags:
    - mac
    - man
---

man是类unix系统最重要的手册工具，安装中文文档有助于我们更好的了解命令行下的各个指令。

<!--more-->

[man中文文档下载地址](https://code.google.com/archive/p/manpages-zh/downloads)

## 安装

### 下载安装包

```bash
wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/manpages-zh/manpages-zh-1.5.1.tar.gz
```

### 解压

```bash
tar zxf manpages-zh-1.5.1.tar.gz
```

### 安装

```bash
cd manpages-zh-1.5.1
# 不需要繁体中文版本
./configure --disable-zhtw 
make && make install
```

### 配置alias

在用户目录中编辑.bash_profile，添加一个cman别名，

> alias cman='man -M /usr/local/share/man/zh_CN'

## 中文乱码问题

### 升级groff

因为mac系统默认的groff版本比较老，所以我们需要更新下，

```bash
brew install homebrew/dupes/groff
brew link homebrew/dupes/groff
```

### 配置man.conf

编辑/etc/man.conf文件，

搜索NROFF，替换为
> NROFF preconv -e utf8 | /usr/local/bin/groff -Wall -mtty-char -Tutf8 -mandoc -c

搜索PAGER，替换为
> PAGER /usr/bin/less -isR

## 参考文章
[Mac 安装man命令中文文档](http://www.jianshu.com/p/5e35202fc59c)
[MAC 系统中显示中文MAN手册](https://linux.cn/article-2163-1.html)