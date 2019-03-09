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
tags: 
    - JavaScript
keywords: iife,自执行匿名函数,立即调用函数表达式
description: 自执行匿名函数或立即调用函数表达式我们可能听说过，假设没听说过，也能知道大概是个什么东西。而IIFE，你造么😂，或者他的一些小技巧知道多少...
photos:
---

js大法也玩了好几年。但今天读到一篇文章，提到了一个词：**`IIFE`**，立马傻眼了，这什么鬼...

于是谷歌了一梭...原来就是个自动执行的函数表达式。**`Immediately-Invoked Function Expression`**

一句话概括就是：定义时就会立即执行的  JavaScript 函数。以下来自`mozilla`的权威解释

> 这是一个被称为 自执行匿名函数 的设计模式，主要包含两部分。
> > 第一部分是包围在 圆括号运算符() 里的一个匿名函数，这个匿名函数拥有独立的词法作用域。这不仅避免了外界访问此 IIFE 中的变量，而且又不会污染全局作用域；
> > 第二部分再一次使用 () 创建了一个立即执行函数表达式，JavaScript 引擎到此将直接执行函数。

在 Javascript 中，圆括号**()**是一种运算符，跟在函数名之后，表示调用该函数。比如，`print()`就表示调用`print`函数。

### 话不多说，看代码

```javascript
/**
 * 1. 当一个函数变成IIFE时，其内部的变量不能从外部访问
 */
(function(){
    var name = 'Jack';
})();

// 外部不能访问变量 name
console.log(name); // undefined

/**
 * 2. 将 IIFE 分配给一个变量，不是存储 IIFE本身，而是存储其执行后返回的结果
 */
var result = (function(){
    var name = 'Jack';
    return name + ' love Rose';
})();

// 此时，IIFE就是一个变量，而不是函数
console.log(result); // Jack love Rose
```

### 结论及用途


- 需要有括号包起来，且两个IIFE连着写时，需要加分号；
- 省略函数命名，避免污染全局变量；
- IIFE内部形成单独的作用域，可以封装一些外部无法读取的私有变量；

#### 一个小知识点

```javascript
// case1
for (var i = 0; i < 5; i++) {
    setTimeout(function() {
        console.log(i);
    }, 1000 * i)
}
// case2
for (let i = 0; i < 5; i++) {
    setTimeout(function() {
        console.log(i);
    }, 1000 * i)
}
```

上面的case1和case2，结果分别是什么？为什么？在看下面：

```javascript
// case3
for (var i = 0; i < 5; i++) {
    (function(i) {
        setTimeout(function() {
            console.log(i);
        }, 1000 * i);
    })(i);
}
```

> case2和case3均打出：0,1,2,3,4。而case1，则：5,5,5,5,5


嗯，神奇现象之：let-for搭配...块作用域的应用


