---
layout:     post
title:      "tensorflow save and restore"
subtitle:   "tensorflow保存和加载"
date:       2019-02-22
author:     "msg"
header-img: "img/posts/01.jpg"
header-mask: 0.3
catalog:    true

tags:
    - tensorflow
    - ubuntu
    - 学习
    - 转载

---

> [Tensorflow加载预训练模型和保存模型](https://blog.csdn.net/huachao1001/article/details/78501928)

### 1 Tensorflow模型文件

在checkpoint_dir目录下保存的文件结构如下：

```
|--checkpoint_dir
|    |--checkpoint
|    |--MyModel.meta
|    |--MyModel.data-00000-of-00001
|    |--MyModel.index
```

#### 1.1 meta文件

MyModel.meta文件保存的是图结构，meta文件是pb（protocol buffer）格式文件，包含变量、op、集合等。

#### 1.2 ckpt文件

ckpt文件是二进制文件，保存了所有的weights、biases、gradients等变量。在tensorflow 0.11之前，保存在.ckpt文件中。0.11后，通过两个文件保存,如：

```
MyModel.data-00000-of-00001
MyModel.index
```

#### 1.3 checkpoint文件

checkpoint文件是个文本文件，里面记录了保存的最新的checkpoint文件以及其它checkpoint文件列表。在inference时，可以通过修改这个文件，指定使用哪个model

### 2 保存Tensorflow模型

tensorflow 提供了tf.train.Saver类来保存模型，在tensorflow中，变量是存在于Session环境中，因此，保存模型时需要传入session：

```python
saver = tf.train.Saver()
saver.save(sess,"./checkpoint_dir/MyModel")
```

一个简单例子：

```python
import tensorflow as tf

w1 = tf.Variable(tf.random_normal(shape=[2]), name='w1')
w2 = tf.Variable(tf.random_normal(shape=[5]), name='w2')
saver = tf.train.Saver()
sess = tf.Session()
sess.run(tf.global_variables_initializer())
saver.save(sess, './checkpoint_dir/MyModel')
```

执行后，在checkpoint_dir目录下创建模型文件如下：

```
checkpoint
MyModel.data-00000-of-00001
MyModel.index
MyModel.meta
```

另外，如果想要在1000次迭代后，再保存模型，只需设置global_step参数即可：

```
saver.save(sess, './checkpoint_dir/MyModel',global_step=1000)
```

保存的模型文件名称会在后面加-1000,如下：

```
checkpoint
MyModel-1000.data-00000-of-00001
MyModel-1000.index
MyModel-1000.meta
```

在实际训练中，可能会在每1000次迭代中保存一次模型数据，但是由于图是不变的，没必要每次都去保存，可以通过如下方式指定不保存图：

```python
saver.save(sess, './checkpoint_dir/MyModel',global_step=step,write_meta_graph=False)
```

如果希望每2小时保存一次模型，并且只保存最近的5个模型文件：

```python
tf.train.Saver(max_to_keep=5, keep_checkpoint_every_n_hours=2)
```

**注意：tensorflow默认只会保存最近的5个模型文件，如果希望保存更多，可以通过max_to_keep来指定**

如果只保存一部分变量，可以通过指定variables/collections。在创建tf.train.Saver实例时，通过将需要保存的变量构造list或者dictionary，传入到Saver中：

```python
import tensorflow as tf
w1 = tf.Variable(tf.random_normal(shape=[2]), name='w1')
w2 = tf.Variable(tf.random_normal(shape=[5]), name='w2')
saver = tf.train.Saver([w1,w2])
sess = tf.Session()
sess.run(tf.global_variables_initializer())
saver.save(sess, './checkpoint_dir/MyModel',global_step=1000)
```

### 3 导入训练好的模型

在导入模型时，要分为2步：**构造网络图和加载参数**

#### 3.1 构造网络图

一个比较笨的方法是，手敲代码，实现跟模型一模一样的图结构。既然已经保存了图，就没必要在去手写一次图结构代码。

```python
saver=tf.train.import_meta_graph('./checkpoint_dir/MyModel-1000.meta')
```

上面一行代码，就把图加载进来了

#### 3.2 加载参数

仅仅有图并没有用，需要前面训练好的模型参数（即weights、biases等），在加载参数时，先要构造好Session：

```python
import tensorflow as tf
with tf.Session() as sess:
  new_saver = tf.train.import_meta_graph('./checkpoint_dir/MyModel-1000.meta')
  new_saver.restore(sess, tf.train.latest_checkpoint('./checkpoint_dir'))
```

此时，W1和W2加载进了图，并且可以被访问：

```python
import tensorflow as tf
with tf.Session() as sess:    
    saver = tf.train.import_meta_graph('./checkpoint_dir/MyModel-1000.meta')
    saver.restore(sess,tf.train.latest_checkpoint('./checkpoint_dir'))
    print(sess.run('w1:0'))
```

执行后，打印如下：

```
[ 0.51480412 -0.56989086]
```

### 4 使用恢复的模型

获取训练好的模型中的一些中间结果值，可以通过graph.get_tensor_by_name('w1:0')来获取，注意w1:0是tensor的name。

假设有一个简单的网络模型，代码如下：

```python
import tensorflow as tf

w1 = tf.placeholder("float", name="w1")
w2 = tf.placeholder("float", name="w2")
b1= tf.Variable(2.0,name="bias") 
w3 = tf.add(w1,w2)
w4 = tf.multiply(w3,b1,name="op_to_restore")
sess = tf.Session()
sess.run(tf.global_variables_initializer())
# 创建一个Saver对象，用于保存所有变量
saver = tf.train.Saver()
# 通过传入数据，执行op
print(sess.run(w4,feed_dict ={w1:4,w2:8}))
# 打印 24.0 ==>(w1+w2)*b1
# 现在保存模型
saver.save(sess, './checkpoint_dir/MyModel',global_step=1000)

```

接下来使用graph.get_tensor_by_name()方法来操纵这个保存的模型。

```python
import tensorflow as tf

sess=tf.Session()
#先加载图和参数变量
saver = tf.train.import_meta_graph('./checkpoint_dir/MyModel-1000.meta')
saver.restore(sess, tf.train.latest_checkpoint('./checkpoint_dir'))
#访问placeholders变量，并且创建feed-dict来作为placeholders的新值

graph = tf.get_default_graph()
w1 = graph.get_tensor_by_name("w1:0")
w2 = graph.get_tensor_by_name("w2:0")
feed_dict ={w1:13.0,w2:17.0}

#接下来，访问你想要执行的op
op_to_restore = graph.get_tensor_by_name("op_to_restore:0")

print(sess.run(op_to_restore,feed_dict))
#打印结果为60.0==>(13+17)*2
```

**注意：保存模型时，只会保存变量的值，placeholder里面的值不会被保存**

如果不仅仅是用训练好的模型，还要加入一些op，或者说加入一些layers并训练新的模型，可以通过一个简单例子来看如何操作：

```python
import tensorflow as tf
sess = tf.Session()
#先加载图和变量
saver = tf.train.import_meta_graph('my_test_model-1000.meta')
saver.restore(sess, tf.train.latest_checkpoint('./'))
#访问placeholders变量，并且创建feed-dict来作为placeholders的新值
graph = tf.get_default_graph()
w1 = graph.get_tensor_by_name("w1:0")
w2 = graph.get_tensor_by_name("w2:0")
feed_dict = {w1: 13.0, w2: 17.0}
#接下来，访问你想要执行的op
op_to_restore = graph.get_tensor_by_name("op_to_restore:0")
#在当前图中能够加入op
add_on_op = tf.multiply(op_to_restore, 2)
print (sess.run(add_on_op, feed_dict))
#打印120.0==>(13+17)*2*2
```

如果只想恢复图的一部分，并且再加入其它的op用于fine-tuning。
只需通过graph.get_tensor_by_name()方法获取需要的op，并且在此基础上建立图。

看一个简单例子，假设需要在训练好的VGG网络使用图，并且修改最后一层，将输出改为2，用于fine-tuning新数据：

```python
saver = tf.train.import_meta_graph('vgg.meta')
# 访问图
graph = tf.get_default_graph() 
# 访问用于fine-tuning的output
fc7= graph.get_tensor_by_name('fc7:0')
# 如果你想修改最后一层梯度，需要如下
fc7 = tf.stop_gradient(fc7)
fc7_shape= fc7.get_shape().as_list()
new_outputs=2
weights = tf.Variable(tf.truncated_normal([fc7_shape[3], num_outputs], stddev=0.05))
biases = tf.Variable(tf.constant(0.05, shape=[num_outputs]))
output = tf.matmul(fc7, weights) + biases
pred = tf.nn.softmax(output)
```