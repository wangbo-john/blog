title: iOS关于指针定义
entitle: 'iOS-Pointer'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528013230
date: 2018-06-03 16:07:10
tags:
    - iOS
    - Objc
keywords: iOS, 指针
description: 来自2015.6.6的笔记：关于指针定义（解决 sending 'const NSString *' to parameter of type 'NSString *' ）
photos:
---


### 关于指针定义（解决 sending 'const NSString *' to parameter of type 'NSString *' ）

```
比如，写了 const NSString* firstString = @"xxx";
NSString* secondString = @"yyy";
[secondString isEqualToString:firstString];

会出现 sending 'const NSString *' to parameter of type 'NSString *' discards qualifiers 警告。
解决办法：
把 const NSString* firstString = @"xxx";
改成 NSString* const firstString = @"xxx";
```

解释：前者相当于指针本身不可修改，后者表示指针指向的内容不可修改，两者的作用都是使firstString只可读不可写。

