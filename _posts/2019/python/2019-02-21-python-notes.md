---
layout:     post
title:      "python notes"
subtitle:   "python笔记"
date:       2019-02-21
author:     "msg"
header-img: "img/posts/04.jpg"
header-mask: 0.3
catalog:    true

tags:
    - python
    - 笔记
    - 学习

---

### jupyter error(404)

#### 安装

```bash

pip install jupyter_nbextensions_configurator jupyter_contrib_nbextensions

jupyter contrib nbextension install --user

jupyter nbextensions_configurator enable --user

```

#### 卸载


```bash

pip uninstall jupyter_nbextensions_configurator jupyter_contrib_nbextensions

jupyter contrib nbextension uninstall --user

jupyter nbextensions_configurator disable --user

```
