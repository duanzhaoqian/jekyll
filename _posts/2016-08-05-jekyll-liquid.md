---
layout: post
title:  "jekyll-liquid"
date:   2016-08-05
desc: "jekyll-liquid遇到的问题"
keywords: "jekyll-liquid遇到的问题"
categories: [Linux]
tags: [jekyll,liquid]
icon: fa-jekyll
---

# jekyll-liquid

<!--
create time: 2016-08-05 17:40:01
Author: <TODO: 请写上你的名字>

This file is created by Marboo<http://marboo.io> template file $MARBOO_HOME/.media/starts/default.md
本文件由 Marboo<http://marboo.io> 模板文件 $MARBOO_HOME/.media/starts/default.md 创建
-->

[liquid api语法](http://alfred-sun.github.io/blog/2015/01/10/jekyll-liquid-syntax-documentation/)

## Raw（解决语法冲突）

临时禁止执行 Jekyll Tag 命令，在生成的内容里存在冲突的语法片段的情况下很有用。

```
{% raw %}
去掉% }间的空格
{ % raw % }
  In Handlebars, {{ this }} will be HTML-escaped, but {{{ that }}} will not.
{ % endraw % }

{% endraw %}
```
