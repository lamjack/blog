---
title: Doctrine2使用utf8mb4
date: 2014-12-31 00:00:00
tags:
	- doctrine2
	- php
	- symfony2
---
在Symfony2配置文件**app/config/config.yml**中找到doctrine配置的地方，

```yaml
# ...
# Doctrine Configuration
doctrine:
    dbal:
        default_connection: default
        connections:
            default:
                # ...
                charset: utf8mb4
                default_table_options:
                    charset: utf8mb4
                    collate: utf8mb4_unicode_ci
                # ...
# ...
```
然后实体的配置文件，注意这里分string类型和text类型两种情况，

```yaml
Acme\DemoBundle\Entity\Hello:
    type: entity
    table: hello
    repositoryClass: Acme\DemoBundle\Repository\HelloRepository
    id:
        openId:
            type: string
            length: 128
            id: true
    fields:
        nickname:
            type: string
            columnDefinition: VARCHAR(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL
            nullable: true
        description:
            type: text
            columnDefinition: LONGTEXT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci
            nullable: true
```

其中，注明type是为了app/console doctrine:generate:entities的时候，实体类知道用什么变量类型。