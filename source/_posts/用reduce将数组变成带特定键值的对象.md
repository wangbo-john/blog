title: 用reduce将数组变成带特定键值的对象
entitle: 'reduce-array-to-object'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1559047234
date: 2019-05-28 20:40:34
tags:
    - JavaScript
keywords: reduce，数组
description: 偶然间看到同事的代码，用reduce把一个字符串数组变成了一个带有以数组元素为Key的Object。如此骚操作，让我不得不重新看看reduce...
photos:
---

### Reduce

出自**ES6**的Array，用来做累加的。以前我一直用来做算术求和，直到前一阵子看到了一个骚操作，于是又做了一番折腾...

看这段代码先：

```
const names = ['jack', 'pony', 'tony', 'thomas'];
const entry = names.reduce((prev, cur) => {
    prev[cur] = `My full name is ${cur} ma.`;
    return prev;
}, {});
```

结果：

```json
{ jack: 'My full name is jack ma.',
  pony: 'My full name is pony ma.',
  tony: 'My full name is tony ma.',
  thomas: 'My full name is thomas ma.' }
```

把一个数组转成了对象，如此优雅的操作。试想，不用这个如何达到效果！？

```
const names = ['jack', 'pony', 'tony', 'thomas'];
const obj = {};
for (let name of names) {
    obj[name] = `My full name is ${name} ma.`
}
```

此时，多定义了一个变量，用了一层循环，看起来很没科技感。戳到API，我们可以看到它是这么定义的：

```typescript
reduce(callbackfn: (previousValue: T, currentValue: T, currentIndex: number, array: T[]) => T, initialValue: T): T;
```

是的，第二个参数竟然是泛型。之前一直传数字做累加，今天记住了。reduce功能不止于此。

