---
title: thymeleaf 常见问题
date: 2020-05-21 21:47:43
author: 东京易冷
categories: thymeleaf
tags:
  - thymeleaf
  - layui
---

## thymeleaf 常见问题

org.thymeleaf.exceptions.TemplateProcessingException: Could not parse as expression

在使用 layui 的 table 模块时，可能会出现如上错误，原因是 [[...]] 会被认为是内联表达式，引起渲染报错，解决方案是，换行处理，如下

```
[
    [
        // 内容
    ]
]
```

参考 https://blog.csdn.net/ystyaoshengting/article/details/84773952