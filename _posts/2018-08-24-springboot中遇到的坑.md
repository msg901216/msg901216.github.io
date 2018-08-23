---
layout:     post
title:      "springboot (1)"
subtitle:   "填坑记录"
date:       2018-08-23
author:     "msg"
header-img: "img/posts/01.jpg"
header-mask: 0.3
catalog:    true

tags:
    - java
    - 学习
    - springboot
---

> 工作中用到springboot，学习springboot的过程中遇到很多坑，下面记录一下填坑过程。（持续记录）

## 时间类型返回时间戳

> 这个问题是jackson将时间类型转换为json格式的时候，默认转为时间戳格式，要想自己控制时间输出的格式，可以进行如下的设置：

* 代码配置方式
```java
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        MappingJackson2HttpMessageConverter jackson2HttpMessageConverter = new MappingJackson2HttpMessageConverter();
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
        jackson2HttpMessageConverter.setObjectMapper(objectMapper);
        converters.add(0,jackson2HttpMessageConverter);
    }
}
```

* 配置文件方式(application.yml)
```
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
```

## 返回时间缺少了8个小时

> 通过controller返回的数据，时间少了八个小时！！！

* 代码配置方式
```java
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        MappingJackson2HttpMessageConverter jackson2HttpMessageConverter = new MappingJackson2HttpMessageConverter();
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setTimeZone(TimeZone.getTimeZone("GMT+8"));
        jackson2HttpMessageConverter.setObjectMapper(objectMapper);
        converters.add(0,jackson2HttpMessageConverter);
    }
}
```

* 配置文件
```
spring:
  jackson:
    time-zone: GMT+8
```