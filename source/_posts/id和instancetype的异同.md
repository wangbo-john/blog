title: id和instancetype的异同
entitle: 'instancetype-iOS'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528012736
date: 2018-06-03 15:58:56
tags:
    - iOS
    - Objc
keywords: iOS, id和instancetype的异同
description: 来自2015.5.13的笔记：id和instancetype的异同。
photos:
---

1、相同点

都可以作为方法的返回类型

2、不同点

①instancetype可以返回和方法所在类相同类型的对象，id只能返回未知类型的对象；
②instancetype只能作为返回值，不能像id那样作为参数，比如下面的写法：

```
//err,expected a type  
- (void)setValue:(instancetype)value  
{  
    //do something  
}  

```

就是错的，应该写成：

```
- (void)setValue:(id)value  
{  
    //do something  
}  
```


