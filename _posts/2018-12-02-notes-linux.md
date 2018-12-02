---
layout:     post
title:      "linux下的一些常用命令记录"
subtitle:   "从ls开始"
date:       2018-12-02
author:     "msg"
header-img: "img/posts/05.jpg"
header-mask: 0.3
catalog:    true

tags:
    - linux
    - ubuntu
    - 学习
    - 转载
---

### 拷贝ssh

```shell
xclip -sel clip < file 
```

### 解压7zip

#### 1.安装

```shell
sudo apt-get install p7zip
```

#### 2.压缩 

```shell
7zr a xxx.7z  foldername
```

#### 3.解压缩

```shell
7zr x xxx.7z
```

### 排序文本内容

```shell
sort text.txt > text-sorted.txt
```

### 删除命令

* dd:删除游标所在的一整行(常用)

* ndd:n为数字。删除光标所在的向下n行，例如20dd则是删除光标所在的向下20行

* d1G:删除光标所在到第一行的所有数据

* dG:删除光标所在到最后一行的所有数据

* d$:删除光标所在处，到该行的最后一个字符

* d0:那个是数字0,删除光标所在到该行的最前面的一个字符

* x,X:x向后删除一个字符(相当于[del]按键),X向前删除一个字符(相当于[backspace]即退格键)

* nx:n为数字，连续向后删除n个字符

### wps for linux 不能使用搜狗输入法

ubuntu版本：18.04
中文输入法：搜狗

#### wps文字不能输入中文解决

```bash
$ vi /usr/bin/wps      # 添加内容，字体标注
#!/bin/bash
export XMODIFIERS="@im=fcitx"
export QT_IM_MODULE="fcitx"
gOpt=
#gOptExt=-multiply
gTemplateExt=("wpt" "dot" "dotx")

```

#### wps表格不能输入中文解决

```bash
$ vi /usr/bin/et      # 添加内容，字体标注
#!/bin/bash
export XMODIFIERS="@im=fcitx"
export QT_IM_MODULE="fcitx"
gOpt=
#gOptExt=-multiply
```

#### 原因：环境变量未正确设置，以上可以直接针对wps设置。

