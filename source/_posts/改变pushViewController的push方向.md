title: 改变pushViewController的push方向
entitle: 'pushViewController-change'
author: 唐先森
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528013902
date: 2018-06-03 16:18:22
tags:
    - iOS
    - Objc
keywords: iOS, pushViewController
description: 来自2015.6.18的笔记：改变pushViewController的push方向
photos:
---

```
CATransition* transition = [CATransition animation];

transition.type = kCATransitionPush;//可更改为其他方式

transition.subtype = kCATransitionFromTop;//可更改为其他方式   

[self.navigationController.view.layera ddAnimation:transition forKey:kCATransition];

[self.navigationController pushViewController:userLogin animated:NO];

```

再来

```
//可根据上一个页面，来确定当前页以何种方式消失
- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
    [self clear];
    BOOL fromRight = YES;
    NSArray *viewControllers = self.navigationController.viewControllers;
    if ([[viewControllers lastObject] isKindOfClass:[BFEAddContactViewController class]]) {
        fromRight = NO;
    }
    CATransition *transition = [CATransition animation];
    transition.type = kCATransitionPush;
    //页面卸载时，改变PUSH方向
    transition.subtype = fromRight ?  kCATransitionFromRight : kCATransitionFromLeft;
    transition.duration = 0.3;
    transition.delegate = self;
    [self.navigationController.view.layer addAnimation:transition forKey:kCATransition];
}
```


