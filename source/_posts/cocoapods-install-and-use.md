---
title: CocoaPods的安装和使用
date: 2016-11-10 14:35:58
categories:
  - 编程语言
  - Objective-C
---
本文适用于Mac OS环境，并默认已经安装了[brew](http://brew.sh/index_zh-cn.html)作为系统组件的管理工具。

# CocoaPods的安装

## 更新Ruby和Gem

在安装CocoaPods之前，首先要在本地安装好Ruby环境。

```bash
brew update
brew install ruby
```

国内原因，建议替换掉RubyGems的源，使用Ruby China的国内镜像，

```bash
gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
# 列出当前源，请确保只有gems.ruby-china.org
gem sources -l
```

## 安装CocoaPods

```bash
gem install cocoapods
```

同样原因，替换成国内镜像，

```bash
pod repo remove master
pod repo add master https://git.coding.net/CocoaPods/Specs.git
pod setup
```

> 在执行移除master命令时，如果报错 [!] repo master does not exist，不用理会，直接执行第二条命令。

> 如果执行第二条命令时报错：[!] To setup the master specs repo, please run pod setup，此时放弃执行以上三条命令，通过以下方法解决。

```bash
git clone https://git.coding.net/CocoaPods/Specs.git ~/.cocoapods/repos/master
```

# CocoaPods的使用

【参考文章】
[Mac OS X 上安装 Ruby](https://github.com/ruby-china/homeland/wiki/Mac-OS-X-%E4%B8%8A%E5%AE%89%E8%A3%85-Ruby)
[CocoaPods的安装和使用](http://www.jianshu.com/p/effda8147b53)
[用CocoaPods做iOS程序的依赖管理](http://blog.devtang.com/2014/05/25/use-cocoapod-to-manage-ios-lib-dependency/)
[解决CocoaPods各种慢的方案](http://hyichao.github.io/ios/2015/12/06/cocoapods-slow.html)