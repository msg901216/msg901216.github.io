---
layout:     post
title:      "rabbitmq notes"
subtitle:   "rabbitmq笔记"
date:       2019-04-24
author:     "msg"
header-img: "img/posts/03.jpg"
header-mask: 0.3
catalog:    true

tags:
    - rabbitmq
    - 笔记
    - 学习


---

#### 安装Erlang

RabbitMQ的安装需要Erlang的基础环境，必须按照[RabbitMQ Erlang版本要求](https://link.juejin.im?target=https%3A%2F%2Fwww.rabbitmq.com%2Fwhich-erlang.html)进行安装。

关于Erlang官方的安装方式有三种：

- 官方制作的[依赖软件包](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Frabbitmq%2Ferlang-rpm)
- [Erlang Solutions的软件包](https://link.juejin.im?target=https%3A%2F%2Fwww.erlang-solutions.com%2Fresources%2Fdownload.html)（这个可以自定义yum库安装，本人自己下载安装）
- [EPEL](https://link.juejin.im?target=http%3A%2F%2Ffedoraproject.org%2Fwiki%2FEPEL)（“Enterprise Linux的额外软件包”）

个人是按照Erlang Solutions的方式安装；根据个人的Linux系统情况选择如下：


![image](https://user-gold-cdn.xitu.io/2018/5/14/1635a9c2f0b3c692?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



基本命令如下：

```
cd /usr/local/src/  
mkdir rabbitmq  
cd rabbitmq  

//下载rpm，如果下载速度慢可以本地下载上传Linux中也可  
wget https://packages.erlang-solutions.com/erlang/esl-erlang/FLAVOUR_1_general/esl-erlang_19.3-1~centos~6_i386.rpm  

rpm –import http://packages.erlang-solutions.com/rpm/erlang_solutions.asc  //导入公钥
  
yum install esl-erlang_19.3-1~centos~6_i386.rpm //安装自动更新依赖（不建议使用rpm安装）

erl //验证是否安装成功
复制代码
```



![安装成果测试](https://user-gold-cdn.xitu.io/2018/5/14/1635a9c2f0ac9a7e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



#### 安装rabbitmq

[所有的安装版本](https://link.juejin.im?target=https%3A%2F%2Fdl.bintray.com%2Frabbitmq%2Fall%2Frabbitmq-server%2F)都在这里了，可以自行根据自己的要求选择安装即可。


![版本说明](https://user-gold-cdn.xitu.io/2018/5/14/1635a9c2f0c43819?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



```
//下载
wget https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.6.6/rabbitmq-server-3.6.6-1.el6.noarch.rpm
//安装
rpm -ivh --nodeps rabbitmq-server-3.6.6-1.el6.noarch.rpm

复制代码
```

## chkconfig rabbitmq-server on

### 注意事项：

1、如果出现如下错误，但是的确安装了erlang对应版本：

```
error: Failed dependencies:
    erlang >= R16B-03 is needed by rabbitmq-server-3.6.3-1.noarch  
复制代码
```

解决方式：

> 添加--nodeps你的rpm命令,参考连接：[StackOverflow问答](https://link.juejin.im?target=https%3A%2F%2Fstackoverflow.com%2Fa%2F40218299%2F877813)

```
rpm -ivh --nodeps rabbitmq-server-3.5.7-1.noarch.rpm
复制代码
```

检查https://stackoverflow.com/a/40218299/877813，并添加--nodeps你的rpm命令

2、出现如下错误的情况;说缺少socat依赖

```
 socat is needed by rabbitmq-server-3.6.6-1.el6.noarch
复制代码
```

解决方式：[参考博客](https://link.juejin.im?target=https%3A%2F%2Fblog.csdn.net%2Fyunfeng482%2Farticle%2Fdetails%2F72853983)

> yum -y install socat
>  此时会报错没有socat包或是找不到socat包，解决方法安装centos的epel的扩展源
>  yum -y install epel-release
>  之后执行yum -y install socat

#### 启动和配置

1.常用命令：

```
//常用的rabbitmq的命令
service rabbitmq-server   start
service rabbitmq-server   stop
service rabbitmq-server   status
service rabbitmq-server   rotate-logs|
service rabbitmq-server   restart
service rabbitmq-server   condrestart
service rabbitmq-server   try-restart
service rabbitmq-server   reload
service rabbitmq-server   force-reload

ps -ef | grep rabbitmq  查看rabbitMq进程

netstat -anplt | grep LISTEN  rabbitmq默认监听端口15672/5672
复制代码
```

2、基本配置：

```
//开启管理页面插件
rabbitmq-plugins enable rabbitmq_management
复制代码
```

管理插件安装完成后，出现如下提示,表示安装成。

```
The following plugins have been enabled:
  mochiweb
  webmachine
  rabbitmq_web_dispatch
  amqp_client
  rabbitmq_management_agent
  rabbitmq_management
Plugin configuration has changed. Restart RabbitMQ for changes to take effect.
复制代码
```

可以用浏览器输入localhost：15672,账号密码全输入guest即可登录：


![image](https://user-gold-cdn.xitu.io/2018/5/14/1635a9c2f0cd0773?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

 测试安装结果图如下：

![image](https://user-gold-cdn.xitu.io/2018/5/14/1635a9c2f0dd67e4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



#### 补充说明部分：

```
//查看linux位数
ls /   #如果有lib64或这个目录,那操作系统就是64位的
getconfig LONG_BIT  若输出32即为32位系统，64为64位系统
 32位的系统中int类型和long类型一般都是4字节，
64位的系统中int类型还是4字节的，但是long已变成了8字节。
inux系统中可 用"getconf WORD_BIT"和"getconf LONG_BIT"获得word和long的位数。
64位系统中应该分别得到32和64。
uname -a中若为X86示意为64位系统，i386等位32位系统
```

