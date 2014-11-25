---
layout: docs
category : note
tagline: "Language Notes"
tags : [jekyll, Language, code, notes]
---
    
本文记录了使用Jekyll中的笔记

## Overview

## Jekyll 环境搭建 （Kali Linux)

1.安装 Jekyll
{% highlight bash %}
gem install jekyll
{% endhighlight %}
2.`clone`  jekyll 模版 到 codeliker.github.io 文件夹中：
{% highlight bash %}
git clone https://github.com/plusjade/jekyll-bootstrap.git  codeliker.github.io
{% endhighlight %}
3.本地测试效果:`jekyll server`，然后浏览器访问`http://127.0.0.1:4000/ 即可

4.如果根目录下同时存在`index.html`和`index.md`文件，`index.html`优先级更高


###标记语言（makedown,md)

1.几个`#`号代表几级标题，颜色为红色

2.两个`TAB`代表代码

3.换行需要打两个`Enter`(submit情况下)