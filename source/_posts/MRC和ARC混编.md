title: MRC和ARC混编
entitle: 'mrc-and-arc'
author: 唐先森
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528012490
date: 2018-06-03 15:54:50
tags:
    - iOS
    - Objc
keywords: iOS, MRC, ARC
description: 来自2015.5.7的笔记：MRC和ARC混编设置。
photos:
---

从XCode5以后，默认都采用了ARC，但有时候又想使用MRC，无奈写了MRC语法后，编译器保持：

![](/img/2018/15280126158577.jpg)

解决方式如下：

![](/img/2018/15280126834131.jpg)

> 注意：-fno-objc-arc 这句不要有空格。

MRC工程中也可以使用ARC的类。方法如下：

在targets的build phases选项下Compile Sources下选择要使用arc编译的文件，双击它，输入 -fobjc-arc 即可


