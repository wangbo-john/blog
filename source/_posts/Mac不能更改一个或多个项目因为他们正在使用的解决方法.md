title: Mac不能更改一个或多个项目因为他们正在使用的解决方法
entitle: 'extended-attri-in-mac'
author: 唐先森
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528020192
date: 2018-06-03 18:03:12
tags:
    - Mac必备工具
    - 黑科技
keywords: macOS, extended-file-attributes
description: 来自2016.5.27的笔记：Mac 不能更改一个或多个项目,因为他们正在使用的解决方法
photos:
    - /img/2018/15280203394758.jpg
---

#### OS X 系统有的文件移动或复制时，出现“Mac 不能更改一个或多个项目,因为他们正在使用”，这或许是多了个属性导致：

![](/img/2018/15280203394758.jpg)


这就解决了，还有，如果是移动设备的文件导致的原因，解决时必须保证移动设备处于可读写状态，对于大部分移动硬盘都是NTFS的问题，可参考如下方法解决：

[https://coolestguidesontheplanet.com/how-to-write-to-ntfs-external-disk-drives-from-os-x-10-11-el-capitan/](https://coolestguidesontheplanet.com/how-to-write-to-ntfs-external-disk-drives-from-os-x-10-11-el-capitan/)


关于@的解释请参考：

[http://mackuba.eu/2008/06/30/ls-on-mac-and-extended-file-attributes/](http://mackuba.eu/2008/06/30/ls-on-mac-and-extended-file-attributes/)


