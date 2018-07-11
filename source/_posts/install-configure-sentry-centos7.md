---
title: CentOS7下部署Sentry(不使用Docker）
date: 2018-07-11 18:18:10
categories:
  - Linux
tags: 
  - centos7
  - sentry
---

本文讲介绍Sentry（一个开源的实时错误报告工具）在CentOS7下如何搭建。安装使用的是epel库，如果你没有安装，请先运行。

```bash
yum install epel-release -y
```

<!--more-->

## 开始安装

1. 更新本机yum组件

```bash
yum update
```

2. 安装所有需要的依赖组件

```bash
yum install wget python-setuptools.noarch python2-pip.noarch python-devel.x86_64 libxslt.x86_64 libxslt-devel.x86_64 libxml2 libxml2-devel.x86_64 libzip libzip-devel libffi.x86_64 libffi-devel.x86_64 openssl-libs.x86_64 libpqxx libpqxx-devel libyaml libyaml-devel libjpeg libjpeg-devel libpng libpng12 libpng12-devel libpng-devel net-tools gcc gcc-c++
```

3. Sentry使用的是Postgresql，

首先安装Postgresql，

```bash
yum install postgresql-server.x86_64 postgresql-contrib
```

初始化，设置开机运行并启动进程，

```bash
postgresql-setup initdb
systemctl enable postgresql.service
systemctl start postgresql.service
```

4. 安装并运行redis

```bash
yum install redis
systemctl enable redis.service
systemctl start redis.service
```

5. 安装supervisor，**这里先不要启动守护进程，稍后我们还需要写一些配置文件**

```bash
yum install supervisor
systemctl enable supervisord.service
```

6. 更新pip

```bash
pip install --upgrade pip
```

7. 安装virtualenv

```bash
pip install -U virtualenv
```

8. 创建Sentry所需要的数据库

```bash
su - postgres
psql template1
create user sentry with password '密码';
alter user sentry with superuser;
create database sentrydb with owner sentry;
\q
exit
```
9. 添加一个用户sentry，用于sentry的管理

```bash
useradd sentry
```

10. 切换到sentry用户

```bash
su - sentry
```

11. 创建一个Python的Virtual Eenvironment，并切换到这个环境下

```bash
virtualenv /data/sentry
source /data/sentry/bin/activate 
```

12. 安装sentry，**这里加U的意思是如果已经安装就更新**

```bash
pip install -U sentry
```

13. 初始化sentry

```bash
/data/sentry/bin/sentry init
```

14. 更新sentry的配置文件，这里只列举关键的部分，具体配置可以参考下官方文档，比如邮件服务器等...

/home/sentry/.sentry/sentry.conf.py

```python
DATABASES = {
	'default': {
		'ENGINE': 'sentry.db.postgres',
		'NAME': 'sentrydb',
		'USER': 'sentry',
		'PASSWORD': 'your_password',
		'HOST': '127.0.0.1',
		'PORT': '5432',
		'AUTOCOMMIT': True,
		'ATOMIC_REQUESTS': False,
	}
}
```

/home/sentry/.sentry/config.yml

```yaml
redis.clusters:
	default:
		hosts:
			0:
				host: 127.0.0.1
				port: 6379
```

15. 更新 pg_hba.conf 文件，并重启Postgresql服务

/var/lib/pgsql/data/pg_hba.conf

```text
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local    all             postgres                                peer
# "local" is for Unix domain socket connections only
local    all             all                                     peer
# IPv4 local connections:
host     all             all             127.0.0.1/32            md5
# IPv6 local connections:
host     all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
#local   replication     postgres                                peer
#host    replication     postgres        127.0.0.1/32            ident
#host    replication     postgres        ::1/128                 ident
```

重启，

```bash
systemctl restart postgresql.service
```

16. 运行Sentry upgrade命令，这里会询问一些问题，并创建一个管理账号

```bash
/home/sentry/sentry_app/bin/sentry upgrade
exit
```

17. 修改 supervisor 配置文件 **/etc/supervisord.conf**

```text
...
files = supervisord.d/*.conf
...
```

然后把 sentry 的 supervisor 配置放到 **/etc/supervisord.d** 路径中，内容如下，

```
[program:sentry-web]
directory=/data/sentry/
environment=SENTRY_CONF="/home/sentry/.sentry"
command=/data/sentry/bin/sentry --config=/home/sentry/.sentry run web
autostart=true
autorestart=true
redirect_stderr=true
user=sentry
stdout_logfile=syslog
stderr_logfile=syslog

[program:sentry-worker]
directory=/data/sentry/
environment=SENTRY_CONF="/home/sentry/.sentry"
command=/data/sentry/bin/sentry --config=/home/sentry/.sentry run worker
autostart=true
autorestart=true
redirect_stderr=true
user=sentry
stdout_logfile=syslog
stderr_logfile=syslog
startsecs=1
startretries=3
stopsignal=TERM
stopwaitsecs=10
stopasgroup=false
killasgroup=true

[program:sentry-cron]
directory=/data/sentry/
environment=SENTRY_CONF="/home/sentry/.sentry"
command=/data/sentry/bin/sentry --config=/home/sentry/.sentry run cron
autostart=true
autorestart=true
redirect_stderr=true
user=sentry
stdout_logfile=syslog
stderr_logfile=syslog

[group:sentry]
programs=sentry-web,sentry-worker,sentry-cron
```

启动 supervisor 进程，

```bash
systemctl start supervisord.service
```

这里我给 sentry 的几个进程分了一个组，如果需要重启 sentry 的所有进程，执行下面命令即可。

```bash
supervisorctl restart sentry:*
```

现在可以通过 http://127.0.0.1:9000 来访问 Sentry 了。

## 使用HTTPS

如果使用 nginx，想要使用强制 https 访问，配置如下，

```nginx
upstream sentry {
    keepalive 1024;
    server 0.0.0.0:9000 max_fails=2 fail_timeout=10m;
}

server {
	listen 80;
	return 301 https://$host$request_uri;
}

server {
    listen      443 ssl;
    server_name yoursentry.com;

    ssl_certificate      /etc/nginx/ssl/yoursentry.crt;
    ssl_certificate_key  /etc/nginx/ssl/yoursentry.key;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:128m;
    ssl_session_timeout 10m;

    location / {
		proxy_pass         http://sentry;
    	proxy_set_header   Host                 $http_host;
    	proxy_set_header   X-Forwarded-Proto    $scheme;
    	proxy_set_header   X-Forwarded-For      $remote_addr;
    	proxy_redirect     off;

    	# keepalive + raven.js is a disaster
    	keepalive_timeout 30;

   		proxy_read_timeout 10s;
    	proxy_send_timeout 10s;
    	send_timeout 10s;
    	resolver_timeout 10s;
    	client_body_timeout 10s;

    	# buffer larger messages
    	client_max_body_size 10m;
    	client_body_buffer_size 100k;

    	add_header Strict-Transport-Security "max-age=31536000";
    }
}
```

## 疑难解决

1. Sentry所使用的 svg 图片浏览器不显示，打开直接下载文件？

首先检查**nginx配置**中的 **mime.types** 是否有这一行，没有的话加上。

```nginx 
types {
    ...
    image/svg+xml                         svg svgz;
    ...
}
```

尝试不要通过反向代理，使用**curl -I**查看资源访问的**Content-Type**，如果不是**image/svg+xml**，比如返回**application/octet-stream**，可以用以下两个方法解决，

指定**ussgi**的**mime.types**路径，

```bash
uwsgi --ini uwsgi.ini --mimefile /etc/mime.types
```

安装**mailcap**来产生**/etc/mime.types**文件（推荐使用此方法）

```bash
yum install mailcap -y
```

