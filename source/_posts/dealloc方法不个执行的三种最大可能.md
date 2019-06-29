title: dealloc方法不个执行的三种最大可能
entitle: 'dealloc-not-work'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528019847
date: 2018-06-03 17:57:27
tags:
    - 苹果
    - objective-c
keywords: iOS, dealloc
description: 来自2016.4.19的笔记：dealloc方法不个执行的三种最大可能
photos:
---

今天写代码时需要在dealloc里移除所有的通知,但是却发现控制器pop后不执行dealloc方法.

> 查到这句话:The dealloc method was not being called if any of the references held by a viewcontroller were still in memory.

dealloc方法没有被调用是因为控制器的一个或多个强引用仍然在内存中,也就是说当前控制器的计数器不为0.
一般的原因有以下几种:

1. 定制器没有被销毁. 
    解决方法:在viewWillDisappear之前需要把控制器用到的NSTimer销毁.

2. block块使用不当, 因为block会对方法中的变量自动retain一次, 请检查控制器中block代码.

3. 代理必须得用weak修饰, 用strong强引用会导致计数器加1，无法释放内存.

4. 在getter方法里使用self. 导致死循环



> Block体内使用实例变量也会造成循环引用，使得拥有这个实例的对象不能释放。


