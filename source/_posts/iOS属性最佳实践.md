title: iOS属性最佳实践
entitle: 'property-in-action-iOS'
author: 唐先森
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528014051
date: 2018-06-03 16:20:51
tags:
    - iOS
    - Objc
keywords: iOS, pushViewController
description: 来自2015.7.29的笔记：iOS属性最佳实践
photos:
    - /img/2018/15280142068491.jpg
---

### 一、关于objective-C的属性，常见的有：strong、weak、copy、assign

1. 对于基本数据类型，当然使用assigin；
2. 对于mutable的，一定要使用strong。父控件UI元素也使用strong；
3. 子控件元素使用weak；
4. 不可变的类型，使用copy。（NSString,NSArray,NSDictonary这些一定要用copy）对于mutable的对象，如果是mutable的却定义属性为copy，则往里面加值时可能引起程序崩溃。而对于不可变的使用了strong，则可能引起值改变，这就违背了内存管理语义

### 二、关于getter

![](/img/2018/15280142068491.jpg)

![](/img/2018/15280142123608.jpg)

```
if (_messageTipNumber > 0) {
        self.tipCountLable.frame = CGRectMake(150, (self.frame.size.height - 14)/2, 14, 14);
        self.tipCountLable.layer.cornerRadius = _tipCountLable.frame.size.width/2;
        self.tipCountLable.text = [NSString stringWithFormat:@"%d",_messageTipNumber];
        [self.contentView addSubview:_tipCountLable];
}
```


