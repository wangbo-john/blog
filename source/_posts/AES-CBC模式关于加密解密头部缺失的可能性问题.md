title: AES/CBC模式关于加密解密头部缺失的可能性问题
entitle: 'AES_CBC-iOS'
author: 唐先森
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528016470
date: 2018-06-03 17:01:10
tags:
    - Objc
    - Java
keywords: iOS, 加解密
description: 来自2016.8.16的笔记：AES CBC模式，关于加密后，解密头部缺失的可能性问题
photos:
    - /img/2018/15280165964723.jpg
---

![](/img/2018/15280165964723.jpg)

如果能解密出来，但是有乱码，如下：

![](/img/2018/15280166141249.jpg)

请考虑IV（初始化向量）是不是有误或两个平台下不一样
====
再有，Java端MD5可能直接返回了bytes，并没有最终转成String，注意多平台的区别

![](/img/2018/15280166311066.jpg)



