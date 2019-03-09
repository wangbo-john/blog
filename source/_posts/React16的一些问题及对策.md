title: React16的一些问题及对策
entitle: 'react16-usage'
author: 唐先森
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528027632
date: 2018-06-03 20:07:12
tags:
    - 前端
    - React
    - JavaScript
keywords: ReactNative, React
description: 来自2018.4.23的笔记：React16升级遇到的一些问题及对策
photos:
    - /img/2018/15280277972569.jpg
---

### 1，如果render中的组件以属性的方式引用了当前类的函数，且在constructor中bind这个函数，会造成这个函数仅刷新一次，如：

![](/img/2018/15280277890966.jpg)

![](/img/2018/15280277972569.jpg)

![](/img/2018/15280278031260.jpg)

#### 解决方式：在Render中bind，在SCU中再拦截，控制实际Render次数

![](/img/2018/15280278204992.jpg)

##  原因是：暂时不知道！！！ 

捂脸.png
捂脸.png
捂脸.png


