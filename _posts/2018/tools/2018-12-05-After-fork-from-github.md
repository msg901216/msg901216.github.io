---
layout:     post
title:      "After fork from github"
subtitle:   "fork"
date:       2018-12-05
author:     "msg"
header-img: "img/posts/01.jpg"
header-mask: 0.3
catalog:    true

tags:
    - fork
    - github
    - 学习
    - 转载

---

> 来自[保持fork之后的项目和上游同步](https://github.com/staticblog/wiki/wiki/%E4%BF%9D%E6%8C%81fork%E4%B9%8B%E5%90%8E%E7%9A%84%E9%A1%B9%E7%9B%AE%E5%92%8C%E4%B8%8A%E6%B8%B8%E5%90%8C%E6%AD%A5)

保持自己fork之后的仓库与上游仓库同步（github为例）。

点击 fork 到自己帐号下，比如[HanLP](https://github.com/hankcs/HanLP)这个仓库:

![Selection_033](/home/msg/Pictures/Selection_033.png)

然后就可以在自己的帐号下 clone 相应的仓库

`git clone https://github.com/algteam/HanLP.git`

使用 `git remote -v` 查看当前的远程仓库地址，输出如下：

```
origin	https://github.com/algteam/HanLP.git (fetch)
origin	https://github.com/algteam/HanLP.git (push)
```

可以看到从自己帐号 clone 下来的仓库，远程仓库地址是与自己的远程仓库绑定的

接下来运行 `git remote add upstream https://github.com/hankcs/HanLP.git`

这条命令就算添加一个别名为 upstream（上游）的地址，指向之前 fork 的原仓库地址。`git remote -v` 输出如下：

```
origin	https://github.com/algteam/HanLP.git (fetch)
origin	https://github.com/algteam/HanLP.git (push)
upstream	https://github.com/hankcs/HanLP.git (fetch)
upstream	https://github.com/hankcs/HanLP.git (push)
```

之后运行下面几条命令，就可以保持本地仓库和上游仓库同步了

```
git fetch upstream
git checkout master
git merge upstream/master
```

接着就是熟悉的推送本地仓库到远程仓库

```
git push origin master
```