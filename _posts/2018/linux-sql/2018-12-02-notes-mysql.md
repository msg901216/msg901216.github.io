---
layout:     post
title:      "mysql的一些常用命令记录"
subtitle:   "mysql commands"
date:       2018-12-02
author:     "msg"
header-img: "img/posts/04.jpg"
header-mask: 0.3
catalog:    true

tags:
    - mysql
    - ubuntu
    - 学习
    - 转载
---

### incorrect string value:'\xf0\x9f

* 在mysql的安装目录下找到my.ini,作如下修改：

```shell
[mysqld]

character-set-server=utf8mb4

[mysql]

default-character-set=utf8mb4
```

修改后重启Mysql

### 将已经建好的表也转换成utf8mb4

命令：

```shell
alter table TABLE_NAME convert to character set utf8mb4 collate utf8mb4_bin; 
```
