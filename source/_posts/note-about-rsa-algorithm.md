---
title: RSA加密算法（应用篇未完结）
date: 2017-02-26 22:45:52
tags:
    - rsa
---
公钥加密算法是计算机通信安全的基石，保证了加密数据不会被破解。

1977年，由三位数学家Rivest、Shamir、Adleman设计了RSA加密算法。

这是一种非对称加密算法，目前被破解的长度（公开数据）768位，因此可以认为，超过768位的RSA密钥至今未能破解，1024位的是安全的，2048位的是极安全的。

<!--more-->

## 对称加密算法和非对称加密算法

### 对称加密算法

> 1）甲方选择一种加密规则，对信息进行加密；
> 2）乙方使用同一种规则，对信息进行解密；

这种加密规则叫”密钥“，这就是[“对称加密算法”](https://zh.wikipedia.org/zh-cn/%E5%AF%B9%E7%AD%89%E5%8A%A0%E5%AF%86)。

### 非对称加密算法

> 1）乙方生成两把密钥（公钥和私钥）
> 2）甲方获取乙方的公钥，然后用它对信息加密。
> 3）乙方得到加密后的信息，用私钥解密。

公钥加密的信息只有私钥解的开，只要私钥不泄露，通讯就是安全的。

## RSA一些概念

这里解释一些概念，有助于理解下面的openssl工具。
RSA可以做数字签名、密钥交换和数据加密。

### 公钥加密
用途是密钥交换，用户A使用用户B的公钥将少量数据加密发给B，B用自己的私钥解密数据；

### 私钥签名
用途是数字签名，用户A使用自己的私钥将数据的摘要信息加密发给B，B用A的公钥解密摘要信息；

## 工具

### openssl工具

这里有比较详细的[中文文档](http://man.linuxde.net/openssl)，这篇文章是讲RSA加密算法的，所以我们只讨论与RSA有关的相关指令和用法。

产生一个密钥，包括了私钥和公钥，也就是说这个密钥可以用来加密也可以用来解密，

```bash
openssl genrsa -out private.pem 1024
```

提取这个密钥的公钥，

```bash
openssl rsa -in private.pem -pubout -out rsa_public_key.pem
```

使用公钥来加密文件，命令中的pubin表示是用纯公钥加密，

```bash
openssl rsautl -encrypt -in test.txt -inkey rsa_public_key.pem -pubin -out test_encrypt.txt
```

用密钥来解密文件，

```bash
openssl rsautl -decrypt -in test_encrypt.txt -inkey private.pem -out test_decrypt.txt
```

## 参考资料
[证书，密钥，加密，rsa到底是啥？](https://blog.phpgao.com/encryption_decryption.html)
[openssl 非对称加密算法RSA命令详解](http://www.cnblogs.com/gordon0918/p/5363466.html)