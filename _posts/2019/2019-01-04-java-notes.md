---
layout:     post
title:      "java笔记"
subtitle:   "java"
date:       2019-01-04
author:     "msg"
header-img: "img/posts/05.jpg"
header-mask: 0.3
catalog:    true

tags:
    - java
    - 笔记
    - 学习

---

### 统一错误处理
```java
@RestController
@RequestMapping("${server.error.path:${error.path:/error}}")
public class MyErrorController extends AbstractErrorController {

    @Autowired
    public MyErrorController(ErrorAttributes errorAttributes) {
        super(errorAttributes);
    }

    public MyErrorController(ErrorAttributes errorAttributes, List<ErrorViewResolver> errorViewResolvers) {
        super(errorAttributes, errorViewResolvers);
    }

    @Override
    public String getErrorPath() {
        return "/error";
    }

    @RequestMapping
    public ResultBean doHandleError(HttpServletRequest request) {
        Map<String, Object> body = getErrorAttributes(request,false);
        ResultBean resultBean = new ResultBean(ResultBean.ResultCode.OTHER_ERROR);
        resultBean.setMsg((String) body.get("message"));
        return resultBean;
    }
}
```

### elasticsearch-data返回score

```java
return template.query(query, response -> {
        List<BotSentenceWithScore> botSentences = new ArrayList<>();
        SearchHit[] hits = response.getHits().getHits();
        for (SearchHit hit : hits) {
            try {
                BotSentenceWithScore botSentence  = entityMapper.mapToObject(hit.getSourceAsString(), BotSentenceWithScore.class);
                float documentScore = hit.getScore();
                botSentence.setScore(documentScore);
                botSentences.add(botSentence);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return botSentences;
    });
```

### entitymapper

```java
@Configuration
public class EntityMapperConfig {
    @Bean
    public EntityMapper entityMapper(){
        return new DefaultEntityMapper();
    }
}
```

### idea打包可执行jar包

1. 选中Java项目工程名称，在菜单中选择 File->project structure... (快捷键Ctrl+Alt+Shift+S)。

2. 在弹出的窗口中左侧选中"Artifacts"，点击"+"选择jar，然后选择"from modules with dependencies"。

3. 在配置窗口中配置"Main Class"。

4. 配置“Directory for META-INF/MAINFEST.MF”，此项配置的缺省值是：..\src\main\java，需要改成项目根目录。选择“extract to the target JAR”，这样所有依赖的jar包都会放在生成的jar包中。

5. 完成后，点击OK，Apply等按钮，回到IDEA的主菜单，选择“Build - Build Artifacts”下的“Build”或者“Rebuild”即可生成最终的可运行的jar。

### aop处理身份验证token

```java
    private Boolean handleToken(ProceedingJoinPoint pjp){
        Object[] objects = pjp.getArgs();
        if(null != objects && objects.length > 0) {
            Object object = objects[0];

            Map<String,Object> map = (Map<String, Object>) object;

            String token = (String) map.get("token");

            if(null != token){
                return token.equals("123456");
            }
        }

        return false;
    }
```

### es分页返回数据

```java
    List<BotSentence> botSentenceList = new ArrayList<>();
            ScrolledPage<BotSentence> botSentenceScrolledPage = (ScrolledPage<BotSentence>) template.startScroll(TimeValue.timeValueMinutes(1).millis(), searchQuery, BotSentence.class);
            while (botSentenceScrolledPage.hasContent()) {
                botSentenceList.addAll(botSentenceScrolledPage.getContent());
                botSentenceScrolledPage = (ScrolledPage<BotSentence>) template.continueScroll(botSentenceScrolledPage.getScrollId(), TimeValue.timeValueMinutes(1).millis(), BotSentence.class);
            }
    template.clearScroll(botSentenceScrolledPage.getScrollId());
```

### springboot连接elasticsearch连接不上

```java
@Component
public class ElasticSearchConfiguration implements InitializingBean {

    static {
        System.setProperty("es.set.netty.runtime.available.processors", "false");
    }

    @Override
    public void afterPropertiesSet() {
        log.info("*****************es_config*************************");
        log.info("es.set.netty.runtime.available.processors:{}", System.getProperty("es.set.netty.runtime.available.processors"));
        log.info("***************************************************");
    }
}
```
