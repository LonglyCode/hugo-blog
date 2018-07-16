---
title: python进阶知识总览
date: 2016-01-21 17:52:11
tags: ["python", "note"]
---
<!-- toc -->

> 有些人学完python之后不知道能干些什么或者要干些什么，实际上python本身语言很容易入门，如果不是**按需学习**就无法更上一层。正如有人抱怨的如今学完一门成熟的语言其实不难，难的是需要花费很多时间去了解它的生态，比如内置库还有第三方库的用法，以及各种成型的流行的框架的使用。有个网站统计开源库所使用库的流行程度，链接在这里[python流行库排行](http://www.programcreek.com/python/index/module/list)。我也是通过这个网站才想到自己有了解过许多的库但是总是零零洒洒，不成体系，所以个人归纳分类了一下，决定每个库都要去实践和记录一下，好记性不如烂笔头。感兴趣的可以关注[我的博客](http://longlycode.github.io/)，会尽量把坑填完。

<!--more-->
## 语法和用法

> *   闭包和装饰器
> *   魔术方法
> *   描述符
> *   生成器和协程
> *   python编码

## 标准库

实际上有许多标准库在学python的过程中已经有所了解，但是总有那么几个标准库本身看起来好像平常没什么用,但到处有在使用，以下几个个人觉得有必要去了解去深入的，当然又要涉及到各种python2.x和python3.x之争，也会在文中提及。

> 1.  logging
> 2.  itertools
> 3.  functools
> 4.  collections
> 5.  contextlib
> 6.  re
> 7.  unittest
> 8.  threading

## 第三库和框架

优秀的库不仅在于它本身的创意新颖或者框架先进，它们的文档一般都非常优秀容易阅读，所以最好的学习方式还是读官方文档。第三库只会提及在某些领域已经被广泛应用的或认可，以下库按用途和功能将其分类。

### 开发环境和调试

> 1.  virtualenv
> 2.  pip
> 3.  ipython 和 ipdb

### 爬虫以及WEB请求和解析

> 1.  scrapy
> 2.  requests
> 3.  Beautiful Soup或者lxml
> 4.  httpie
> 5.  html2text

### 数据库相关

> 1.  SQLAlchemy

### 渲染模板引擎

> 1.  Jinja2

### 网站框架

> 1.  Flask
> 2.  Djiango

### 异步和协程

> 1.  gevent
> 2.  Celery
