title: iOS页面间几种传值方式
entitle: 'data-pass-in-iOS'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528013544
date: 2018-06-03 16:12:24
tags:
    - 苹果
    - objective-c
keywords: iOS, 属性，代理，block，单例，通知
description: 来自2015.6.11的笔记：iOS 页面间几种传值方式（属性，代理，block，单例，通知）
photos:
---

### 属性：

在继承关系下，子类使用父类的数据通过属性最为合适，也最直接明了。

### Block：

如果有某种继承或所属关系时，父元素要使用子元素的数据，那么此时应该使用block回调。因为此时子元素属性就不一定能取到值（初始化未或动作未必完成）。

### 消息：

两个类根本没有关联，则可以采取发消息的方式。如果两个类可以引入某一方，则还是采取属性或者block方法，因为发消息实际上通过KVO比较消耗系统资源。能不发消息，尽量不要发消息。


