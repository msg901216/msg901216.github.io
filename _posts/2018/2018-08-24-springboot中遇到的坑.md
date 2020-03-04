---
layout:     post
title:      "2018-08-23 springboot (1)"
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

## 1、时间类型返回时间戳

> 这个问题是jackson将时间类型转换为json格式的时候，默认转为时间戳格式，要想自己控制时间输出的格式，可以进行如下的设置：

* 代码配置方式
```java
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        MappingJackson2HttpMessageConverter jackson2HttpMessageConverter = new 
        MappingJackson2HttpMessageConverter();
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

## 2、返回时间缺少了8个小时

> 通过controller返回的数据，时间少了八个小时！！！

* 代码配置方式
```java
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        MappingJackson2HttpMessageConverter jackson2HttpMessageConverter = new 
        MappingJackson2HttpMessageConverter();
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

## 3、对项目中的error的返回数据做统一处理

```java
@RestController
@RequestMapping("${server.error.path:${error.path:/error}}")
public class MyErrorController implements ErrorController {
    @Override
    public String getErrorPath() {
        return "/error";
    }

    @RequestMapping
    public ResultBean doHandleError() {
        return new ResultBean(ResultCode.TIMEOUT);
    }
}
```

## 4、对项目中的exception做统一处理

1) 第一种方式：

```java

@ControllerAdvice
@ResponseBody
public class ExceptionHandlerAdvice {

    @ExceptionHandler(Exception.class)
    public ResultBean handleException(Exception e) {
        e.printStackTrace();
        return new ResultBean(ResultCode.OTHER_ERROR);
    }

    @ExceptionHandler(NoNodeAvailableException.class)
    public ResultBean handleNoNodeAvailableException(NoNodeAvailableException e) {
        return new ResultBean(ResultCode.ES_TIMEOUT);
    }
}

```

2) 第二种方式：

> @Aspect 注解；\\
> 植入点：\\
> 方法返回值为：ResultEntity\\
> 所有带有controller层级的包 下面的 所有类的所有方法

```java
@Aspect
@Component
public class ControllerAOP {

    @Pointcut("execution(public * com.msg.*.controller.*.*(..))")
    public void webLog() {
    }

    @Around("webLog()")
    public Object handlerControllerMethod(ProceedingJoinPoint pjp) {
        long startTime = System.currentTimeMillis();
        ResultBean<?> result;
        try {
            result = (ResultBean<?>) pjp.proceed();
            long stopTime = System.currentTimeMillis();
            log.info(pjp.getSignature() + "use time:" + ( stopTime- startTime));
        } catch (Throwable e) {
            result = handlerException(pjp, e);
        }

        return result;
    }

    private ResultBean<?> handlerException(ProceedingJoinPoint pjp, Throwable e) {
        ResultBean<?> result;
        e.printStackTrace();
        // 已知异常
        if (e instanceof BadSqlGrammarException) {
            result = new ResultBean<>(ResultCode.ILLEGAL_SQL);
        } else if (e instanceof HttpMessageNotReadableException) {
            result = new ResultBean<>(ResultCode.ILLEGAL_JSON);
        } else if (e instanceof NoNodeAvailableException) {
            result = new ResultBean<>(ResultCode.ES_TIMEOUT);
        }else if(e instanceof BatchInsertException){
            result = new ResultBean<>(ResultCode.EMPTY_LIST);
        }else if(e instanceof ClassCastException){
            result = new ResultBean<>(ResultCode.WRONG_PARAM);
        }


        else {
            result = new ResultBean<>(ResultCode.OTHER_ERROR);
        }

        return result;
    }
}
```

## 5、跨域配置

```java
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("POST", "PUT", "GET", "DELETE")
                .allowCredentials(false).maxAge(3600);
    }
}
```

## 6、springboot获取sessionId
```java
public String getSessionId(){
    HttpSession httpSession = getSession();
    String id = httpSession.getId();
    return id;
}


private HttpSession getSession() {
    HttpSession session = null;
    try {
        session = getRequest().getSession();
    } catch (Exception e) {
        log.error("获取session失败！");
    }
    return session;
}

private HttpServletRequest getRequest() {
    ServletRequestAttributes attrs = (ServletRequestAttributes) RequestContextHolder
            .getRequestAttributes();
    return attrs.getRequest();
}
```

