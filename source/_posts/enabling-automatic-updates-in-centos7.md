---
title: 让CentOS7自动升级YUM组件
date: 2016-10-10 15:47:16
tags:
  - centos
---
### yum 安装 yum-cron
```bash
yum install yum-cron -y
```

### 配置

打开配置文件 /etc/yum/yum-cron.conf

```config
#  What kind of update to use:
# default                            = yum upgrade
# security                           = yum --security upgrade
# security-severity:Critical         = yum --sec-severity=Critical upgrade
# minimal                            = yum --bugfix upgrade-minimal
# minimal-security                   = yum --security upgrade-minimal
# minimal-security-severity:Critical =  --sec-severity=Critical upgrade-minimal
update_cmd = default

# Whether a message should be emitted when updates are available,
# were downloaded, or applied.
update_messages = yes

# Whether updates should be downloaded when they are available.
download_updates = yes

# Whether updates should be applied when they are available.  Note
# that download_updates must also be yes for the update to be applied.
apply_updates = no

# Maximum amout of time to randomly sleep, in minutes.  The program
# will sleep for a random amount of time between 0 and random_sleep
# minutes before running.  This is useful for e.g. staggering the
# times that multiple systems will access update servers.  If
# random_sleep is 0 or negative, the program will run immediately.
# 6*60 = 360
random_sleep = 360
```

将 update_messages, download_updates, apply_updates 这几个配置项修改为 yes。

### 设置开机自启动并启动服务
```bash
systemctl enable crond
systemctl enable yum-cron
systemctl start crond
systemctl start yum-cron
```

