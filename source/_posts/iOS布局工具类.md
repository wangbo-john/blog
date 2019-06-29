title: iOS布局工具类
entitle: 'pure-coding-layout-iOS'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528015316
date: 2018-06-03 16:41:56
tags:
    - 苹果
    - objective-c
keywords: iOS, 布局
description: 来自2015.4.25的笔记：iOS手码布局工具类
photos:
---

```
#import <Foundation/Foundation.h>

typedef enum
{
    AlignmentCenter = 0,
    AlignmentTop,
    AlignmentBottom,
    AlignmentLeft,
    AlignmentRight
} PlaceAlignmentType;

@interface PositionTools : NSObject

// 针对默认为竖屏的有效
+ (void)placeView:(UIView*)subview atTheCenterOfTheView:(UIView*)container;

+ (void)placeView:(UIView*)subview atTheLeftMiddleOfTheView:(UIView*)container offset:(float)offset;
+ (void)placeView:(UIView*)subview atTheRightMiddleOfTheView:(UIView*)container offset:(float)offset;
+ (void)placeView:(UIView*)subview atTheTopMiddleOfTheView:(UIView*)container offset:(float)offset;
+ (void)placeView:(UIView*)subview atTheBottomMiddleOfTheView:(UIView*)container offset:(float)offset;

+ (void)placeView:(UIView*)subview atTheLeftTopOfTheView:(UIView*)container offset:(CGSize)offset;
+ (void)placeView:(UIView*)subview atTheRightTopOfTheView:(UIView*)container offset:(CGSize)offset;
+ (void)placeView:(UIView*)subview atTheLeftBottomOfTheView:(UIView*)container offset:(CGSize)offset;
+ (void)placeView:(UIView*)subview atTheRightBottomOfTheView:(UIView*)container offset:(CGSize)offset;

+ (void)placeView:(UIView *)sourceView toTheRightOfTheView:(UIView*)targetView span:(float)span;
+ (void)placeView:(UIView *)sourceView toTheLeftOfTheView:(UIView*)targetView span:(float)span;
+ (void)placeView:(UIView *)sourceView aboveTheView:(UIView*)targetView span:(float)span;
+ (void)placeView:(UIView *)sourceView belowTheView:(UIView*)targetView span:(float)span;

+ (void)placeView:(UIView *)sourceView toTheRightOfTheView:(UIView*)targetView span:(float)span alignmentType:(PlaceAlignmentType)alignmentType;
+ (void)placeView:(UIView *)sourceView toTheLeftOfTheView:(UIView*)targetView span:(float)span alignmentType:(PlaceAlignmentType)alignmentType;
+ (void)placeView:(UIView *)sourceView aboveTheView:(UIView*)targetView span:(float)span alignmentType:(PlaceAlignmentType)alignmentType;
+ (void)placeView:(UIView *)sourceView belowTheView:(UIView*)targetView span:(float)span alignmentType:(PlaceAlignmentType)alignmentType;

// 用来处理横屏的情况下，在屏幕没有旋转到横屏位置之前的情况
+ (void)placeView:(UIView*)subview atTheCenterOfTheView:(UIView*)container needAlter:(BOOL)needAlter;

+ (void)placeView:(UIView*)subview atTheLeftMiddleOfTheView:(UIView*)container offset:(float)offset needAlter:(BOOL)needAlter;
+ (void)placeView:(UIView*)subview atTheRightMiddleOfTheView:(UIView*)container offset:(float)offset needAlter:(BOOL)needAlter;
+ (void)placeView:(UIView*)subview atTheTopMiddleOfTheView:(UIView*)container offset:(float)offset needAlter:(BOOL)needAlter;
+ (void)placeView:(UIView*)subview atTheBottomMiddleOfTheView:(UIView*)container offset:(float)offset needAlter:(BOOL)needAlter;

+ (void)placeView:(UIView*)subview atTheRightTopOfTheView:(UIView*)container offset:(CGSize)offset needAlter:(BOOL)needAlter;
+ (void)placeView:(UIView*)subview atTheLeftBottomOfTheView:(UIView*)container offset:(CGSize)offset needAlter:(BOOL)needAlter;
+ (void)placeView:(UIView*)subview atTheRightBottomOfTheView:(UIView*)container offset:(CGSize)offset needAlter:(BOOL)needAlter;

@end
```

实现

```
#import "PositionTools.h"

@implementation PositionTools

+ (void)placeView:(UIView*)subview atTheCenterOfTheView:(UIView*)container
{
    return [self placeView:subview atTheCenterOfTheView:container needAlter:NO];
}

+ (void)placeView:(UIView*)subview atTheLeftMiddleOfTheView:(UIView*)container offset:(float)offset
{
    return [self placeView:subview atTheLeftMiddleOfTheView:container offset:offset needAlter:NO];
}

+ (void)placeView:(UIView*)subview atTheRightMiddleOfTheView:(UIView*)container offset:(float)offset
{
    return [self placeView:subview atTheRightMiddleOfTheView:container offset:offset needAlter:NO];
}

+ (void)placeView:(UIView*)subview atTheTopMiddleOfTheView:(UIView*)container offset:(float)offset
{
    return [self placeView:subview atTheTopMiddleOfTheView:container offset:offset needAlter:NO];
}

+ (void)placeView:(UIView*)subview atTheBottomMiddleOfTheView:(UIView*)container offset:(float)offset
{
    return [self placeView:subview atTheBottomMiddleOfTheView:container offset:offset needAlter:NO];
}

+ (void)placeView:(UIView*)subview atTheRightTopOfTheView:(UIView*)container offset:(CGSize)offset
{
    return [self placeView:subview atTheRightTopOfTheView:container offset:offset needAlter:NO];
}

+ (void)placeView:(UIView*)subview atTheLeftBottomOfTheView:(UIView*)container offset:(CGSize)offset
{
    return [self placeView:subview atTheLeftBottomOfTheView:container offset:offset needAlter:NO];
}

+ (void)placeView:(UIView*)subview atTheRightBottomOfTheView:(UIView*)container offset:(CGSize)offset
{
    return [self placeView:subview atTheRightBottomOfTheView:container offset:offset needAlter:NO];
}

+ (void)placeView:(UIView*)subview atTheCenterOfTheView:(UIView*)container needAlter:(BOOL)needAlter
{
    [container addSubview:subview];
    
    if (needAlter) {
        subview.center = CGPointMake(container.bounds.size.height/2, container.bounds.size.width/2);
    } else {
        subview.center = CGPointMake(container.bounds.size.width/2, container.bounds.size.height/2);
    }
}

+ (void)placeView:(UIView*)subview atTheLeftMiddleOfTheView:(UIView*)container offset:(float)offset needAlter:(BOOL)needAlter
{
    [container addSubview:subview];
    
    if (needAlter) {
        subview.center = CGPointMake(offset + subview.frame.size.width/2, container.bounds.size.width/2);
    } else {
        subview.center = CGPointMake(offset + subview.frame.size.width/2, container.bounds.size.height/2);
    }
}

+ (void)placeView:(UIView*)subview atTheRightMiddleOfTheView:(UIView*)container offset:(float)offset needAlter:(BOOL)needAlter
{
    [container addSubview:subview];
    
    if (needAlter) {
        subview.center = CGPointMake(container.bounds.size.height - (offset + subview.frame.size.width/2), container.bounds.size.width/2);
    } else {
        subview.center = CGPointMake(container.bounds.size.width - (offset + subview.frame.size.width/2), container.bounds.size.height/2);
    }
}

+ (void)placeView:(UIView*)subview atTheTopMiddleOfTheView:(UIView*)container offset:(float)offset needAlter:(BOOL)needAlter
{
    [container addSubview:subview];
    
    if (needAlter) {
        subview.center = CGPointMake(container.bounds.size.height/2, offset + subview.frame.size.height/2);
    } else {
        subview.center = CGPointMake(container.bounds.size.width/2, offset + subview.frame.size.height/2);
    }
}

+ (void)placeView:(UIView*)subview atTheBottomMiddleOfTheView:(UIView*)container offset:(float)offset needAlter:(BOOL)needAlter
{
    [container addSubview:subview];
    
    if (needAlter) {
        subview.center = CGPointMake(container.bounds.size.height/2,  container.bounds.size.width - (offset + subview.frame.size.height/2));
    } else {
        subview.center = CGPointMake(container.bounds.size.width/2,  container.bounds.size.height - (offset + subview.frame.size.height/2));
    }
}

+ (void)placeView:(UIView*)subview atTheLeftTopOfTheView:(UIView*)container offset:(CGSize)offset
{
    [container addSubview:subview];
    
    subview.center = CGPointMake(offset.width + subview.frame.size.width/2,
                                 offset.height + subview.frame.size.height/2);
}

+ (void)placeView:(UIView*)subview atTheRightTopOfTheView:(UIView*)container offset:(CGSize)offset needAlter:(BOOL)needAlter
{
    [container addSubview:subview];
    
    if (needAlter) {
        subview.center = CGPointMake(container.bounds.size.height - (offset.width + subview.frame.size.width/2),
                                     offset.height + subview.frame.size.height/2);
    } else {
        subview.center = CGPointMake(container.bounds.size.width - (offset.width + subview.frame.size.width/2),
                                     offset.height + subview.frame.size.height/2);
    }
}

+ (void)placeView:(UIView*)subview atTheLeftBottomOfTheView:(UIView*)container offset:(CGSize)offset needAlter:(BOOL)needAlter
{
    [container addSubview:subview];
    
    if (needAlter) {
        subview.center = CGPointMake(offset.width + subview.frame.size.width/2,
                                     container.bounds.size.width - (offset.height + subview.frame.size.height/2));
    } else {
        subview.center = CGPointMake(offset.width + subview.frame.size.width/2,
                                     container.bounds.size.height - (offset.height + subview.frame.size.height/2));
    }
}

+ (void)placeView:(UIView*)subview atTheRightBottomOfTheView:(UIView*)container offset:(CGSize)offset needAlter:(BOOL)needAlter
{
    [container addSubview:subview];
    
    if (needAlter) {
        subview.center = CGPointMake(container.bounds.size.height - (offset.width + subview.frame.size.width/2),
                                     container.bounds.size.width - (offset.height + subview.frame.size.height/2));
    } else {
        subview.center = CGPointMake(container.bounds.size.width - (offset.width + subview.frame.size.width/2),
                                     container.bounds.size.height - (offset.height + subview.frame.size.height/2));
    }
}

+ (void)placeView:(UIView *)sourceView toTheRightOfTheView:(UIView*)targetView span:(float)span
{
    [self placeView:sourceView toTheRightOfTheView:targetView span:span alignmentType:AlignmentCenter];
}

+ (void)placeView:(UIView *)sourceView toTheLeftOfTheView:(UIView*)targetView span:(float)span
{
    [self placeView:sourceView toTheLeftOfTheView:targetView span:span alignmentType:AlignmentCenter];
}

+ (void)placeView:(UIView *)sourceView aboveTheView:(UIView*)targetView span:(float)span
{
    [self placeView:sourceView aboveTheView:targetView span:span alignmentType:AlignmentCenter];
}

+ (void)placeView:(UIView *)sourceView belowTheView:(UIView*)targetView span:(float)span
{
    [self placeView:sourceView belowTheView:targetView span:span alignmentType:AlignmentCenter];
}

+ (void)placeView:(UIView*)sourceView toTheRightOfTheView:(UIView*)targetView span:(float)span alignmentType:(PlaceAlignmentType)alignmentType
{
    [[targetView superview] addSubview:sourceView];
    sourceView.center = CGPointMake(targetView.frame.origin.x + targetView.frame.size.width + (span + sourceView.frame.size.width/2),
                                    targetView.center.y);
    [self alignView:sourceView targetView:targetView alignmentType:alignmentType];
}

+ (void)placeView:(UIView*)sourceView toTheLeftOfTheView:(UIView*)targetView span:(float)span alignmentType:(PlaceAlignmentType)alignmentType
{
    [[targetView superview] addSubview:sourceView];
    sourceView.center = CGPointMake(targetView.frame.origin.x - (span + sourceView.frame.size.width/2),
                                    targetView.center.y);
    [self alignView:sourceView targetView:targetView alignmentType:alignmentType];
}

+ (void)placeView:(UIView*)sourceView aboveTheView:(UIView*)targetView span:(float)span alignmentType:(PlaceAlignmentType)alignmentType
{
    [[targetView superview] addSubview:sourceView];
    sourceView.center = CGPointMake(targetView.center.x,
                                    targetView.frame.origin.y - (span + sourceView.frame.size.height/2));
    [self alignView:sourceView targetView:targetView alignmentType:alignmentType];
}

+ (void)placeView:(UIView*)sourceView belowTheView:(UIView*)targetView span:(float)span alignmentType:(PlaceAlignmentType)alignmentType
{
    [[targetView superview] addSubview:sourceView];
    sourceView.center = CGPointMake(targetView.center.x,
                                    targetView.frame.origin.y + targetView.frame.size.height + (span + sourceView.frame.size.height/2));
    [self alignView:sourceView targetView:targetView alignmentType:alignmentType];
}

+ (void)alignView:(UIView*)sourceView targetView:(UIView*)targetView alignmentType:(PlaceAlignmentType)alignmentType
{
    if (AlignmentTop == alignmentType) {
        sourceView.center = CGPointMake(sourceView.center.x,
                                        targetView.frame.origin.y + sourceView.frame.size.height/2);
    } else if (AlignmentBottom == alignmentType) {
        sourceView.center = CGPointMake(sourceView.center.x,
                                        targetView.frame.origin.y + targetView.frame.size.height - sourceView.frame.size.height/2);
    } else if (AlignmentLeft == alignmentType) {
        sourceView.center = CGPointMake(targetView.frame.origin.x + sourceView.frame.size.width/2,
                                        sourceView.center.y);
    } else if (AlignmentRight == alignmentType) {
        sourceView.center = CGPointMake(targetView.frame.origin.x + targetView.frame.size.width - sourceView.frame.size.width/2,
                                        sourceView.center.y);
    }
}
@end
```


