---
title: homebrew 服务使用 root 权限运行
date: 2015-01-26 00:00:00
tags:
  - mac
---
**[homebrew](http://brew.sh/index_zh-cn.html "http://brew.sh")** 是使用 MACOS 开发者不可或缺的套件管理器。

我们经常会通过它来安装一个套件，比如 nginx, mysql 等，由于 MAC 系统下如果需要绑定 1024 以下的端口，需要使用 root 权限去启动守护进程，我们可以这样做：

```bash
brew install nginx
sudo chown root:wheel /usr/local/Cellar/nginx/1.6.2/bin/nginx
sudo chmod u+s /usr/local/Cellar/nginx/1.6.2/bin/nginx
```
