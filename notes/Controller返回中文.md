---
tags: [java/基础知识]
title: Controller返回中文
created: '2019-11-27T09:09:57.839Z'
modified: '2019-12-05T10:29:11.358Z'
---

# Controller返回中文

在controller直接返回值给页面的话。 需要用printwriter，并且在Response设置contentType

``` java
@RequestMapping("/getJson")  
public void getJson(HttpServletRequest request,HttpServletResponse response) throws IOException{  
    response.setContentType("application/json;charset=utf-8");  
    PrintWriter out = response.getWriter();  
    String str = "{name:'一些中文',id:'123'}";  
    out.write(str);  
}  
```
