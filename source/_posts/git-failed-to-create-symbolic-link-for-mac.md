---
title: Max下GIT克隆Window代码符号链接符不一致导致的错误解决
date: 2017-04-20 11:10:20
categories:
  - 疑难杂症
tags:
  - git
---
最近克隆一个github上的项目时候，出现了一个错误。

> ln: failed to create symbolic link "xxx" File name too long

这里记录下我的解决方式。

```bash
$ git clone https://github.com/xxx/xxx.git
$ cd xxx
$ git config --list
$ git config core.symlinks false
$ git checkout master
```

参考文章：

[Git下Windows与Linux符号链接兼容](https://www.jianshu.com/p/05b777fa22bc)

[Git config coresymlinks](https://git-scm.com/docs/git-config#git-config-coresymlinks)