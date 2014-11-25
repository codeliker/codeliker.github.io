---
layout: docs
category : note
tagline: "introduce to git usage when using github"
tags : [intro, beginner, environment, tutorial]
---
解决`jekyll` 使用 `Sumlime Text` 无法正确提取摘要的问题，这个是由`jekyll`默认的`excerpt_separator`即编辑器引起的。

`jekyll`默认的`excerpt_separator`值为`\n\n`，而`sublime`的换行符为\x0d\x0a，这样是无法匹配的。解决办法：指定`excerpt_separator`的值使之可以匹配，如`\n`即可。