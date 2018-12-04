---
layout:     post
title:      "springboot 学习笔记"
subtitle:   "springboot notes"
date:       2018-12-02
author:     "msg"
header-img: "img/posts/04.jpg"
header-mask: 0.3
catalog:    true

tags:
    - springboot
    - java
    - 学习
    - 转载
---

## 1 springboot @Valid注解对输入的参数进行校验

限制 说明 
@Null 限制只能为null

@NotNull 限制必须不为null 

@AssertFalse 限制必须为false 

@AssertTrue 限制必须为true 

@DecimalMax(value) 限制必须为一个不大于指定值的数字 

@DecimalMin(value) 限制必须为一个不小于指定值的数字 

@Digits(integer,fraction) 限制必须为一个小数，且整数部分的位数不能超过integer，小数部分的位数不能超过fraction 

@Future 限制必须是一个将来的日期 

@Max(value) 限制必须为一个不大于指定值的数字 

@Min(value) 限制必须为一个不小于指定值的数字 

@Pattern(value) 限制必须符合指定的正则表达式 

@Size(max,min) 限制字符长度必须在min到max之间 

@Past 验证注解的元素值（日期类型）比当前时间早 

@NotEmpty 验证注解的元素值不为null且不为空（字符串长度不为0、集合大小不为0）

@NotBlank 验证注解的元素值不为空（不为null、去除首位空格后长度为0），不同于@NotEmpty，@NotBlank只应用于字符串且在比较时会去除字符串的空格 

@Email 验证注解的元素值是Email，也可以通过正则表达式和flag指定自定义的email格式 

## 2 对项目中的error的返回数据做统一处理

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

## 3 对项目中的exception做统一处理

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

    @ExceptionHandler(HttpMessageNotReadableException.class)
    public ResultBean handleHttpMessageNotReadableException(HttpMessageNotReadableException e) {
        return new ResultBean(ResultCode.ILLEGAL_JSON);
    }

    @ExceptionHandler(NoNodeAvailableException.class)
    public ResultBean handleNoNodeAvailableException(NoNodeAvailableException e) {
        return new ResultBean(ResultCode.ES_TIMEOUT);
    }
}

```

2) 第二种方式：

> @Aspect 注解；

> 织入点：

> 方法返回值为：ResultEntity

> 所有带有controller层级的包 下面的 所有类的所有方法

```java
@Around("execution(public com.ssslinppp.model.ResultEntity com...controller...*(..))")
@Component
@Aspect
public class ControllerAspect {
    public static final Logger logger = LoggerFactory.getLogger(ControllerAspect.class);

    @Around("execution(public com.ssslinppp.model.ResultEntity com..*.controller..*.*(..))")
    public Object handleControllerMethod(ProceedingJoinPoint pjp) {
        Stopwatch stopwatch = Stopwatch.createStarted();

        ResultEntity<?> resultEntity;
        try {
            logger.info("执行Controller开始: " + pjp.getSignature() + " 参数：" + Lists.newArrayList(pjp.getArgs()).toString());
            resultEntity = (ResultEntity<?>) pjp.proceed(pjp.getArgs());
            logger.info("执行Controller结束: " + pjp.getSignature() + "， 返回值：" + resultEntity.toString());
            logger.info("耗时：" + stopwatch.stop().elapsed(TimeUnit.MILLISECONDS) + "(毫秒).");
        } catch (Throwable throwable) {
            resultEntity = handlerException(pjp, throwable);
        }

        return resultEntity;
    }

    private ResultEntity<?> handlerException(ProceedingJoinPoint pjp, Throwable e) {
        ResultEntity<?> resultEntity = null;
        if (e instanceof RuntimeException) {
            logger.error("RuntimeException{方法：" + pjp.getSignature() + "， 参数：" + pjp.getArgs() + ",异常：" + e.getMessage() + "}", e);
            resultEntity = ResultEntity.fail(e.getMessage());
        } else {
            logger.error("异常{方法：" + pjp.getSignature() + "， 参数：" + pjp.getArgs() + ",异常：" + e.getMessage() + "}", e);
            resultEntity = ResultEntity.fail(e.getMessage());
        }

        return resultEntity;
    }
}

```

