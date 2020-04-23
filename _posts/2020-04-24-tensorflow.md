---
layout:     post
title:      "tensorflow function notes"
subtitle:   "function notes"
date:       2020-04-24
author:     "msgi"
header-img: "img/posts/02.jpg"
header-mask: 0.3
catalog:    true

tags:
    - tensorflow
    - AI
    - 笔记
    - 学习
---

```python
squeeze(
    input,
    axis=None,
    name=None,
    squeeze_dims=None
)
```
### 001、tf.squeeze

该函数返回一个张量，这个张量是将原始input中所有维度为1的那些维都删掉的结果

axis可以用来指定要删掉的为1的维度，此处要注意指定的维度必须确保其是1，否则会报错

```python
y = tf.squeeze(inputs, [0, 1], name='squeeze')
>>>ValueError: Can not squeeze dim[0], expected a dimension of 1, got 32 for 'squeeze' (op: 'Squeeze') with input shapes: [32,1,1,3].
```

例子：

```python
#  't' 是一个维度是[1, 2, 1, 3, 1, 1]的张量
tf.shape(tf.squeeze(t))   # [2, 3]， 默认删除所有为1的维度

# 't' 是一个维度[1, 2, 1, 3, 1, 1]的张量
tf.shape(tf.squeeze(t, [2, 4]))  # [1, 2, 3, 1]，标号从零开始，只删掉了2和4维的1
```
