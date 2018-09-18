---
layout:     post
title:      "python notes"
subtitle:   "Python学习笔记"
date:       2018-09-18
author:     "msg"
header-img: "img/posts/03.jpg"
header-mask: 0.3
catalog:    true

tags:
    - Python
    - 学习
    - 笔记
---

### from __future__ import print_function作用


在开头加上from __future__ import print_function这句之后，即使在python2.X，使用print就得像python3.X那样加括号使用。python2.X中print不需要括号，而在python3.X中则需要。
```python
# python2.7
print "Hello world"

# python3
print("Hello world")
```

如果某个版本中出现了某个新的功能特性，而且这个特性和当前版本中使用的不兼容，也就是它在该版本中不是语言标准，那么我如果想要使用的话就需要从future模块导入。
其他例子： 

```python
from __future__ import division 
from __future__ import absolute_import 
from __future__ import with_statement
 ```
 
等等
加上这些，如果你的python版本是python2.X，你也得按照python3.X那样使用这些函数。
