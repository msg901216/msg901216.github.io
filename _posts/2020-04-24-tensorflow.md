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

### 001、tf.squeeze

```python
squeeze(
    input,
    axis=None,
    name=None,
    squeeze_dims=None
)
```

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


### 002、tf.math.rsqrt

```python
rsqrt(
    x,
    name=None
)
```

tf.rsqrt 函数用于计算 x 元素的平方根的倒数.

$$y = 1 / \sqrt{x}$$

### 003、tf.floor
返回不大于x的元素最大整数。

```python
tf.math.floor(
    x,
    name=None
)
```

### 004、tf.linalg.band_part

> update: 2020-04-28

函数原型：

```python
tf.linalg.band_part(
    input,
    num_lower,
    num_upper,
    name=None
)
```

参数：

作用：主要功能是以对角线为中心，取它的副对角线部分，其他部分用0填充。

input:输入的张量.

num_lower:下三角矩阵保留的副对角线数量，从主对角线开始计算，相当于下三角的带宽。取值为负数时，则全部保留。

num_upper:上三角矩阵保留的副对角线数量，从主对角线开始计算，相当于上三角的带宽。取值为负数时，则全部保留。

例子：

```python
import tensorflow as tf
tf.enable_eager_execution()
a=tf.constant( [[ 1,  1,  2, 3],[-1,  2,  1, 2],[-2, -1,  3, 1],
                 [-3, -2, -1, 5]],dtype=tf.float32)
b=tf.linalg.band_part(a,2,0)
c=tf.linalg.band_part(a,1,1)
d=tf.linalg.band_part(a,-1,1)
print(a)
print(b)
print(c)
print(d)
```

输出：

```bash
tf.Tensor(
[[ 1.  1.  2.  3.]
 [-1.  2.  1.  2.]
 [-2. -1.  3.  1.]
 [-3. -2. -1.  5.]], shape=(4, 4), dtype=float32)
=============================================================
tf.Tensor(
[[ 1.  0.  0.  0.]
 [-1.  2.  0.  0.]
 [-2. -1.  3.  0.]
 [ 0. -2. -1.  5.]], shape=(4, 4), dtype=float32)
 =============================================================
tf.Tensor(
[[ 1.  1.  0.  0.]
 [-1.  2.  1.  0.]
 [ 0. -1.  3.  1.]
 [ 0.  0. -1.  5.]], shape=(4, 4), dtype=float32)
  =============================================================
tf.Tensor(
[[ 1.  1.  0.  0.]
 [-1.  2.  1.  0.]
 [-2. -1.  3.  1.]
 [-3. -2. -1.  5.]], shape=(4, 4), dtype=float32)
————————————————
```
