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

### 使用curl发送请求

**使用curl发送GET请求**

```bash
curl protocol://address:port/url?args
```

例如：

```bash
curl http://127.0.0.1:8080/login?admin&passwd=12345678
```

**使用curl发送POST请求**

```bash
curl -d "args" protocol://address:port/url
```

例如：

```bash
curl -d "user=admin&passwd=12345678" http://127.0.0.1:8080/login
```

这种方法是参数直接在header里面的，如需将输出指定到文件可以通过重定向进行操作

```bash
curl -H "Content-Type:application/json" -X POST -d 'json data' URL
```

例如：

```bash
curl -H "Content-Type:application/json" -X POST -d '{"user": "admin", "passwd":"12345678"}' http://127.0.0.1:8000/login
```

