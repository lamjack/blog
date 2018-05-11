---
title: CentOS编译openssl提示unable to get local issuer certificate解决
date: 2018-05-11 14:15:00
categories:
  - Linux
tags: 
  - centos7
  - openssl
---

CentOS 默认安装的 openssl CA证书在 /etc/pki/tls，编译安装情况下openssl是包括 root CA certificates的。

> The OpenSSL project does not (any longer) include root CA certificates.

```bash
$ openssl s_client -connect baidu.com:443
```

因此会看到 unable to get local issuer certificate 这个错误。

![1](https://jack-images.wilead.net/blog/bta3j.png)

解决方案，

```bash
$ openssl version -a
```

查看 OPENSSLDIR 路径，然后把 CentOS 默认的 openssl CA证书拷贝过来。

```bash
$ cp /etc/pki/tls/cert.pem /usr/local/openssl/ssl
```

问题解决。

![2](https://jack-images.wilead.net/blog/cgk92.png)