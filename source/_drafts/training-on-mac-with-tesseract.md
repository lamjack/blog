---
title: MAC环境下使用tesseract构建自己的训练集
date: 2018-04-07 00:17:41
categories:
  - 机器学习
tags:
  - tesseract
---

Tesseract的OCR引擎最先由HP实验室于1985年开始研发，至1995年时已经成为OCR业内最准确的三款识别引擎之一。然而，HP不久便决定放弃OCR业务，Tesseract也从此尘封。

数年以后，HP意识到，与其将Tesseract束之高阁，不如贡献给开源软件业，让其重焕新生。

2005年，Tesseract由美国内华达州信息技术研究所获得，并求诸于Google对Tesseract进行改进、消除Bug、优化工作。

Tesseract目前已作为开源项目发布在Google Project，其项目主页在这里查看，其最新版本3.0已经支持中文OCR，并提供了一个命令行工具。

下面我们尝试使用Tesseract来训练我们自己的字库。

<!--more-->

## 环境准备

### 安装 tesseract

```bash
$ brew install --with-training-tools tesseract
```

如果已经安装过 tesseract ，但是没有安装训练工具，可以使用 reinstall 重新安装。

### jTessBoxEditor 字库训练工具

jTessBoxEditor 安装之前需要确保本机有最新版的 Java SE Development Kit。

[jTessBoxEditor下载地址](https://sourceforge.net/projects/vietocr/files/jTessBoxEditor/)

下载后解压缩，执行。

```bash
$ java -Xms4096m -Xmx4096m -jar jTessBoxEditor.jar
```

其中 Xms 和 Xmx 分别是负责调整初始堆大小和最大堆大小。

