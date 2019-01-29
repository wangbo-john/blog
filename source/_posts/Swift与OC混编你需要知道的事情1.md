title: Swift与OC混编你需要知道的事情1
entitle: 'swift-oc-coding'
author: 唐先森
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528018777
date: 2018-06-03 17:39:37
tags:
    - iOS
    - Swift
keywords: iOS, swift
description: 来自2016.5.5的笔记：Swift与OC混编你需要知道的事情，第一篇
photos:
    - /img/2018/15280190471314.jpg
---

### 在Swift中调用OC代码

如果是纯OC项目，当你创建第一个Swift文件时，Xcode会提示你建立一个$(PROJECT_NAME)-Bridging-Header.h文件，这个文件就是OC与Swift间相互交流的桥梁文件，即：所有需要在Swift中调用的OC代码，OC头文件必须在这个文件里引入，相反如果是纯Swift项目，当你建立第一个OC语法的文件时，他也会提示，照做就可以了，酱紫就完成了Swift中调用OC。

![](/img/2018/15280190471314.jpg)

如果发现建立了桥接文件而项目无法正常编译时，请检查如上配置

### 在OC中调用Swift代码

由于Swift中没有头文件的概念，所有在OC中，直接引入Swift文件，编译器会不高兴的，结果就是编译无法通过！！！所以你需要在调用Swift的OC代码中，引入一个名叫：$(PROJECT_NAME)-Swift.h，这个文件中定义了该项目中所有Swift的类及其方法，不过他是不可见的

![](/img/2018/15280190820933.jpg)

