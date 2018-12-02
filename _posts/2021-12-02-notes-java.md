---
layout:     post
title:      "java的学习笔记"
subtitle:   "java notes"
date:       2018-12-02
author:     "msg"
header-img: "img/posts/04.jpg"
header-mask: 0.3
catalog:    true

tags:
    - java
    - ubuntu
    - 学习
    - 转载
---

### 更新maven项目版本号

```
mvn versions:set -DnewVersion=2.0.0-SNAPSHOT
```

### @Data注解

原文中提到的大致有以下几点： 

1. 此注解会生成equals(Object other) 和 hashCode()方法。 
2. 它默认使用非静态，非瞬态的属性 
3. 可通过参数exclude排除一些属性 
4. 可通过参数of指定仅使用哪些属性 
5. 它默认仅使用该类中定义的属性且不调用父类的方法 
6. 可通过callSuper=true解决上一点问题。让其生成的方法中调用父类的方法。

另：@Data相当于@Getter @Setter @RequiredArgsConstructor @ToString @EqualsAndHashCode这5个注解的合集。

通过官方文档，可以得知，当使用@Data注解时，则有了@EqualsAndHashCode注解，那么就会在此类中存在equals(Object other) 和 hashCode()方法，且不会使用父类的属性，这就导致了可能的问题。 
比如，有多个类有相同的部分属性，把它们定义到父类中，恰好id（数据库主键）也在父类中，那么就会存在部分对象在比较时，它们并不相等，却因为lombok自动生成的equals(Object other) 和 hashCode()方法判定为相等，从而导致出错。

修复此问题的方法很简单： 

1. 使用@Getter @Setter @ToString代替@Data并且自定义equals(Object other) 和 hashCode()方法，比如有些类只需要判断主键id是否相等即足矣。 
2. 或者使用在使用@Data时同时加上@EqualsAndHashCode(callSuper=true)注解。

### 单点登录

　什么是单点登录？单点登录全称Single Sign On（以下简称SSO），是指在多系统应用群中登录一个系统，便可在其他所有系统中得到授权而无需再次登录，包括单点登录与单点注销两部分。

### 跨域

```java
@Configuration
public class CorsConfiguration {
 
    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurerAdapter() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**")
                        .allowedHeaders("*")
                        .allowedMethods("*")
                        .allowedOrigins("*").allowCredentials(true);
            }
        };
    }
}
```

### Idea突然不停indexing的问题

发现Idea中文件不停的indexing，界面不停闪烁，选择文件选不中等，只要清理一下Idea的缓存和索引就可以了，在File-Invalidate Caches / Restart中，选择Invalidate and Restart，之后会重启Idea，解决！

### java内部类调用外部类

直接使用外部类名称.this.方法即可，例如Inner是内部类，Outer是外部类，在Inner中调用Outer.this.Print()即可使用Outer的print方法

### elastic search排序

org.elasticsearch.search.sort.SortBuilder是一个抽象类，有4个子类

org.elasticsearch.search.sort.FieldSortBuilder 根据某属性值排序

org.elasticsearch.search.sort.GeoDistanceSortBuilder 根据地理位置排序

org.elasticsearch.search.sort.ScoreSortBuilder 根据score排序

org.elasticsearch.search.sort.ScriptSortBuilder 根据自定义脚本排序

可以通过SortBuilders的4个静态方法来分别生成SortBuilder的4个子类实例。可以通过SortBuiler的实例方法order(SortOrder order)来指定是升序还是倒序，默认是升序。

```java
#创建ServiceLog类用来自定义注解
@Retention(RetentionPolicy.Runtime)
@Target(ElementType.METHOD)
public @interface ServiceLog {
 
}
 
#定义ServiceLogAspect类用来定义日志打印信息
 
@Component
@Aspect
public class ServiceLogAspect {
 
   public ThreadLocal<Long> local = new ThreadLocal<Long>();
    
   @Pointcut("@annotation(com.test.XXX.ServiceLong)")
   public void pointCut() {
     
   }
 
   @Before("pointCut()")
   public void before(JoinPoint point) {
    String methodName = point.getTarget().getClass().getName()+"."+point.getSignature().getName();
    local.set(System.currentTimeMillis());
   }
 
  @After("pointCut()")
   public void after(JoinPoint point) {
    long start = local.get();
    String methodName = point.getTarget().getClass().getName()+"."+point.getSignature().getName();
    System.out.println(System.currentTimeMillis()-start));
    }
    
  @AfterThrowing(pointcut="pointCut()",throwing="error")
   public void throwing(JoinPoint point,Throwable error) {
    System.out.println("error");
    }
 
}
```

### 单例模式里双重加锁的机制

```java
public static PhraseSimilarity getInstance() {
        //拿掉判断，直接运行synchronization，会使每个getInstance()都会得到一个静态内部锁，降低了效率
    if (instance == null) {
        synchronized (PhraseSimilarity.class) {
            //同步块内不进行二次检验的话就会生成多个实例了。
            if (instance == null) {
                instance = new PhraseSimilarity();
            }
        }
    }
    return instance;
}
```

