---
layout:     post
title:      "linux 笔记"
subtitle:   "linux"
date:       2019-04-23
author:     "msg"
header-img: "img/posts/03.jpg"
header-mask: 0.3
catalog:    true

tags:
    - linux
    - 笔记
    - 学习

---

### linux 基础操作
```cat /etc/os-release # 查看操作系统信息 ``` 

**突然各种No route to host的原因**

```bash
ip a
route -n
```
查看到多了一个172.20.0.0的网卡,删除掉即恢复了正常

```bash
sudo ifconfig 网卡名 down
```