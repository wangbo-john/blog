title: ReactNative导航的解决方案
entitle: 'react-navigation'
author: 唐先森
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528027186
date: 2018-06-03 19:59:46
tags:
    - 前端
    - ReactNative
keywords: ReactNative, React Navigation
description: 来自2018.2.4的笔记：ReactNative导航的解决方案（Navigating Between Screens）
photos:
---

### 一、纯JS的解决方案：React Navigation

易用、跨平台、良好的状态管理
教程：

> https://www.jianshu.com/p/2f575cc35780
> https://www.jianshu.com/p/b877115fff1b

概况，React Navigation分为三个部分：
```
StackNavigator：类似顶部导航条，用来跳转页面和传递参数；
TabNavigator：类似底部标签，用来区分模块；
DrawerNavigator：抽屉，类似从APP侧滑出一个页面
```

#### 1、注意事项：

> iOS和Android平台的tabs默认行为不一致，表现在iOS正常，安卓的tab在顶部且处于可滑动状态，一统江山方法如下：

```javascript
const Navigation = TabNavigator(
    {
        tab1: { screen: screen1 },
        tab2: { screen: screen2 },
    },
    // customize app tab bars
    {
        tabBarPosition: 'bottom',
        tabBarComponent: TabBarBottom, // 安卓默认是顶部，不设置该项可能导致tabIcon位置错误
        swipeEnabled: false, // 安卓默认可滑动
        lazy: true,
        initialRouteName: 'Offline',
        tabBarOptions: {
            indicatorStyle: {
                height: 0, // android 中TabBar下面会显示一条线，高度设为 0 后就不显示线了
            },
            style: {
                height: 49,
                backgroundColor: 'white'
          },
            labelStyle: {
                marginBottom: 3
            },
            iconStyle: {
                height: 24,
                width: 24,
                margin: 0
            },
            showIcon: true, // 是否显示图标，安卓默认关闭
            // label和icon的前景色 活跃状态下（选中）
            activeTintColor: '#4ECBFC',
            // label和icon的前景色 不活跃状态下(未选中)
            inactiveTintColor: '#aaa',
            // label和icon的背景色 活跃状态下
            activeBackgroundColor: 'white',
            // label和icon的背景色 不活跃状态下
            inactiveBackgroundColor: 'white',
            // 不透明度为按选项卡(iOS和Android < 5.0)
            pressOpacity: 0.3,
            scrollEnabled: false， // 是否启用可滚动的选项卡，安卓特有
        }
    }
);
```

**安卓tabbar文字会下移，因为安卓比iOS多一个属性，就是iconStyle，通过设置labelStyle和iconStyle两个样式，可以调整整理合理性。**

#### 2、安卓导航栏文字默认居左调整
自定义导航样式，重写：

> headerTitleStyle

#### 3、让安卓实现push动画

```javascript
// 先引入这个方法
import CardStackStyleInterpolator from 'react-navigation/src/views/CardStackStyleInterpolator';
​
// 在StackNavigator配置headerMode的地方，使用transitionConfig添加
{
    headerMode: 'screen',
    transitionConfig:()=>({
        screenInterpolator:CardStackStyleInterpolator.forHorizontal,
    })
}

```

### 二、iOS平台专用的：NavigatorIOS

提供UINavigationViewController类的方法。其中Navigator已经被正式废弃！（于RN44被抛弃）

> 替代方案：react-native-deprecated-custom-components

### 三、Native外观和体验的跨端方案：native-navigation, react-native-navigation.

其中前者有AirBnb团队开发，1.x前处于不稳定版本。不建议投入正式用途。
后者由WiX团队提供。目前功能已经相对完整，功能完全。必须在RN 0.43以后才可以用。
如果是混合开发，比如RN模块只作为一个TAB嵌入，那么可以考虑这种方案。

> 具体资料如下：https://wix.github.io/react-native-navigation/#/

