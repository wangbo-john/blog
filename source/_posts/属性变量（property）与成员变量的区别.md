title: 属性变量（property）与成员变量的区别
entitle: 'property-and-memberVar-in-iOS'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528011307
date: 2018-06-03 15:35:07
tags:
    - 苹果
    - objective-c
keywords: iOS, property
description: 来自2015.4.29的笔记：属性变量（property）与成员变量的区别。
photos:
---

```
@interface MyViewController :UIViewControlle
{
    UIButton *myButton;
}
@property (nonatomic, retain) UIButton *myButton;
@end

```

类与类别中添加的属性要区分开来，因为类别中只能添加方法，不能添加实例变量。经常会在ios的代码中看到在类别中添加属性，这种情况下，是不会自动生成实例变量的。比如在：UINavigationController.h文件中会对UIViewController类进行扩展

```
@interface UIViewController (UINavigationControllerItem)
@property(nonatomic,readonly,retain) UINavigationItem *navigationItem;
@property(nonatomic) BOOL hidesBottomBarWhenPushed;
@property(nonatomic,readonly,retain) UINavigationController *navigationController;
@end

```

这里添加的属性，不会自动生成实例变量，这里添加的属性其实是添加的getter与setter方法。

> 注意一点，匿名类别(匿名扩展)是可以添加实例变量的，非匿名类别是不能添加实例变量的，只能添加方法，或者属性（其实也是方法）。

成员变量用于类内部，无需与外界接触的变量。

根据成员变量的私有性，为了方便访问，所以就有了属性变量。属性变量的好处就是允许让其他对象访问到该变量。当然，你可以设置只读或者可写等，设置方法也可自定义。所以，属性变量是用于与其他对象交互的变量。
一些建议:

1. 如果只是单纯的private变量，最好声明在implementation里.
2. 如果是类的public属性，就用property写在.h文件里
3. 如果自己内部需要setter和getter来实现一些东西，就在.m文件的类目里用property来声明


