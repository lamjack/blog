---
title: 验证私钥是否与SSL证书和CSR文件配对
date: 2018-06-27 11:55:19
categories:
  - Linux
tags: 
  - openssl
---

当我们手里管理着多个HTTPS网站时，我们很容易混淆SSL证书或者某个CSR文件是由对应哪个私钥生成的。我们可以通过openssl工具来进行一致性(**Compatibility**)检测。

当我们创建私钥和CSR文件的时候，OpenSSL会生成一个内部的特征值(**modulus**)，当时候这个私钥和CSR文件获取SSL证书的时候，这个特征值也会保存在SSL证书中。

如果私钥和SSL证书的特征值不能匹配的话，会收到这样一个错误：

>  *[error] Unable to configure RSA server Private Key [error] SSL Library Error: x509 certificate routines:X509_check_private_key:key values mismatch*.

所以当我们遇到这个错误的时候，我们就要对私钥和SSL证书的特征值进行校对。

通常我们校对的方法是获取私钥、CSR文件和SSL证书的特征值，然后把它转成md5，

获取SSL证书特征值的md5散列串，

```bash
openssl x509 -noout -modulus -in SSL.crt | openssl md5
```

获取CSR文件特征值的md5散列串，

```bash
openssl req -noout -modulus -in CSR.csr | openssl md5
```

获取私钥特征值的md5散列串，

```bash
openssl rsa -noout -modulus -in PRIVATEKEY.key | openssl md5
```

这是在我本机的一个测试（私钥、CSR文件和证书都是本机签发，用于开发测试使用），

![1](https://jack-images.wilead.net/blog/boevf.png)

如果特征值的md5散列串一致，那么它们是匹配的。