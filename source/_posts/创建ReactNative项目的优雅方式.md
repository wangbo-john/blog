title: 创建ReactNative项目的优雅方式
entitle: 'create-rn-in-better-way'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528026020
date: 2018-06-03 19:40:20
tags:
    - 前端
    - react-native
    - javascript
keywords: ReactNative, npm, nodejs
description: 来自2018.1.19的笔记：创建ReactNative项目的优雅方式
photos:
    - /img/2018/15280262453183.jpg
---

### 先说不优雅的方式，即：react-native init xxx

可以在init时加上模板参数，即：

> react-native init xxx --template youui  // 这个意思就是生成youui开发样板（脚手架）

这种方式会生成XCode及Android Studio工程。通过XCode等工具可以直接运行模拟器或真机。

### 优雅的方式

#### 1、官方推荐的QuickStart，即CRNA：create-react-native-app
![](/img/2018/15280262453183.jpg)

**CRNA方式会生成一个带Expo的环境，但不会有XCode和Android Studio工程。也不会有各种目录配置。通过Expo客户扫描控制台二维码启动应用。
**

#### 2、Ignite生成项目

IGNITE的官方地址：https://github.com/infinitered/ignite

> IGNITE是一个React Native的脚手架生成器（了解ROR的可以理解为rails命令），通过一个命令就可以生成一个结构完整的、可工作的空白react native项目，后续的开发就是向这个项目添砖加瓦，这比从头构建一个RN项目节省很多时间。而且IGNITE默认集成的很多库也都是不二之选，包含了前人的经验。

**ignite new 之后会生成标准的开发结构。包括Reactotron配置等。结构比较重，版本也不见得是最新的！**

#### 3、Expo方式

一、概念
普及英文读法： ['ɛkspoʊ]，本身是展览会的意思。

来看官方的解释：https://docs.expo.io/versions/latest/index.html

> Expo is a set of tools, libraries and services which let you build native iOS and Android apps by writing JavaScript

```
Expo是一组工具、库和服务，可以通过编写JavaScript来构建本地的ios和Android应用程序
Expo Apps是包含了Expo SDK的react native Apps,SDK是一个native-and-js的库，它包提供对设备系统的访问功能，像照相机、联系人、本地存储和其他硬件）。这意味着你不需要使用Xcode或Android的环境，或写任何代码也使得你的pure-JS项目非常便携，因为它可以运行在任何自然环境包含Expo SDK。
Expo还提供UI组件来处理各种应用程序，几乎所有应用程序都将被覆盖，但它不会突破react native Core的核心代码，例如图标、模糊视图，等等。
最后，Expo SDK提供了访问服务，这些服务虽然很难管理，但几乎每个应用程序都需要它。其中最受欢迎的是：Expo可以为您管理您的资产，它可以为您处理推送通知，并且它可以构建准备部署到应用程序商店的本地二进制文件
```

FaceBook的解释如下：

https://facebook.github.io/react-native/docs/more-resources.html Expo is a development environment plus application that focuses on letting you build React Native apps in the Expo development environment, without ever touching Xcode or Android Studio. If you wish React Native was even more JavaScripty and webby, check out Expo.

![](/img/2018/15280264067172.jpg)


二、和Ignite等模板的区别？

Ignite是一个脚手架生成工具，提供便捷的模板。区别于Expo，它没有类似环境、服务的概念。某种意义上跟CRNA是一种东西。而后者主要是提供另一种RN开发环境，比如Windows下开发流程等。Ignite可以作为模板扩展，加入到Expo工程里


![](/img/2018/15280264415058.jpg)

### 关于打造自己的脚手架

https://zhuanlan.zhihu.com/p/32190298

### 另外有一篇关于：React Native App应用架构设计

绝对是干货：[React Native App应用架构设计](https://zhuanlan.zhihu.com/p/30617441)


