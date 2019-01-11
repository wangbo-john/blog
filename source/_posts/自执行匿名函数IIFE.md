title: 自执行匿名函数IIFE
entitle: 'iife-notes'
author: 唐先森
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1546923599
date: 2019-01-08 12:59:59
tags: JavaScript
keywords: iife,自执行匿名函数,立即调用函数表达式
description: 自执行匿名函数或立即调用函数表达式我们可能听说过，假设没听说过，也能知道大概是个什么东西。而IIFE，你造么😂，或者他的一些小技巧知道多少...
photos:
---

js大法也玩了好几年。但今天读到一篇文章，提到了一个词：**`IIFE`**，立马傻眼了，这什么鬼...

于是谷歌了一梭...原来就是个自动执行的函数表达式。一句话概括就是：定义时就会立即执行的  JavaScript 函数。

以下来自`mozilla`的权威解释

> 这是一个被称为 自执行匿名函数 的设计模式，主要包含两部分。
> > 第一部分是包围在 圆括号运算符() 里的一个匿名函数，这个匿名函数拥有独立的词法作用域。这不仅避免了外界访问此 IIFE 中的变量，而且又不会污染全局作用域；
> > 第二部分再一次使用 () 创建了一个立即执行函数表达式，JavaScript 引擎到此将直接执行函数。

### 话不多说，看代码

```javascript
function case1() {
    var name = 'jack';
}

// 更新个东西，稍后接着写...






```


