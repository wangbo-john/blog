title: UIView事件独占
entitle: 'exclusiveTouch-iOS'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528013386
date: 2018-06-03 16:09:46
tags:
    - 苹果
    - objective-c
keywords: iOS, 指针
description: 来自2015.6.10的笔记：UIView事件独占
photos:
---

### UIView 的exclusiveTouch属性

exclusiveTouch的意思是UIView会独占整个Touch事件，具体的来说，就是当设置了exclusiveTouch的 UIView是事件的第一响应者，那么到你的所有手指离开前，其他的视图UIview是不会响应任何触摸事件的，对于多点触摸事件，这个属性就非常重要，值得注意的是：手势识别（GestureRecognizers）会忽略此属性。

列举用途：我们知道ios是没有GridView视图的，通常做法是在UITableView的cell上加载几个子视图，来模拟实现 GridView视图，但对于每一个子视图来说，就需要使用exclusiveTouch，否则当同时点击多个子视图，那么会触发每个子视图的事件。当然 还有我们常说的模态对话框。

