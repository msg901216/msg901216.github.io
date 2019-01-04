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


