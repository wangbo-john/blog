title: Flutter和Rax初探
entitle: 'flutter-and-rax-hello'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1531130418
date: 2018-07-09 18:00:18
tags:
    - flutter
    - 前端
    - 跨端
keywords: flutter, rax
description: 本文对现有的Flutter和Rax跨端框架进行了简单调研，并得出了一些结论。应该是可以解决初学时的What and Why...
photos:
---

## Flutter篇

### 环境安装（macOS）

#### 参考`Flutter`中文网的教程：[https://flutterchina.club/setup-macos/](https://flutterchina.club/setup-macos/)

以上流程基本上是一把梭的就装好了，唯一需要注意的是**PUB_HOSTED_URL**和**FLUTTER_STORAGE_BASE_URL**这俩得写进环境变量配置里，如下：

```
# 1，我直接加到了系统环境变量中，需要加到自己家目录下同理
vim /etc/profile

# 2，复制以下配置
# Flutter SDK https://flutterchina.club/setup-macos/
PUB_HOSTED_URL=https://pub.flutter-io.cn
FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
PATH=~/dev-lib/flutter/bin:$PATH

export PATH PUB_HOSTED_URL FLUTTER_STORAGE_BASE_URL

# 3，使环境变量生效

source /etc/profile

```

#### 校验其他依赖

执行：`flutter doctor`，该命令检查您的环境并在终端窗口中显示报告，所以缺什么，直接复制命名继续安装即可，直到我这样就行了：

![](/img/2018/15311311861503.jpg)

### 开始撸代码，首先配置IDE（此处以VS Code为例）

#### 配置方式如下

[Visual Studio Code (VS Code) Dart安装](https://flutterchina.club/get-started/editor/#vscode)


#### 启动模拟器

![](/img/2018/15311314047697.jpg)

这里用iOS模拟器演示，执行：`flutter emulators --launch apple_ios_simulator`。然后模拟器就被正常启动

#### 打开VS Code并创建Demo工程，按F5后

![Simulator Screen Shot - iPhone 8 Plus - 2018-07-09 at 18.20.54](/img/2018/Simulator%20Screen%20Shot%20-%20iPhone%208%20Plus%20-%202018-07-09%20at%2018.20.54.png)

### 至此，`Flutter`的`Hello-World`工程就跑起来了，之后需要做的就是去学习语法开始撸代码


## Rax篇

> 先来区分一下概念：Weex和Rax
> > 前者是一个容器（运行时环境），后者是一个跨容器的渲染引擎。即：Rax可以跑在Weex里，但Weex不能跑在Rax中。
> > 两者是相互独立的。Weex可以简单理解为：`Vue-Native`。拿知乎段友的话概括就是：“看了一下好像是淘宝写 react-web 那几个人弄的，我猜他们是要用 weex 但是又喜欢 react 那套思想，所以整了个类 react 的东西，可以直接在 weex 里面渲染，也就是说可以用类似 react-native 的方式来写 weex 了”

具体也可以看看这里：[Rax 系列教程（native 扫盲）](http://taobaofed.org/blog/2018/02/06/rax-native-guide/)

### 安装

[Rax快速开始](https://alibaba.github.io/rax/guide/getting-started)

```
# 1. 安装命令行
npm install -g rax-cli

# 2. 初始化项目
rax init hello-rax

# 3. 启动 
npm run start
```

### 跑起来

#### Web端

通过控制台生成的二维码，直接访问

#### Native端

1. 先安装Weex: `npm install -g weex-toolkit`

再在Rax目录中创建Weex环境：

![](/img/2018/15312064317112.jpg)

2. 通过**weex create**创建的工程，没有iOS和Android工程模板，此时需要安装**weex**应用模板

```
// iOS (需要手动执行pod install)
weex platform add ios

// 安卓
weex platform add android
```

3. 打开`WeexDemo.xcworkspace`，修改`BUNDLE_URL`并运行`playground`

![](/img/2018/15312115498098.jpg)

4. 启动模拟器，就可以看到`Rax`跑起来了

![](/img/2018/15312116337280.jpg)



## 总结

1. 大腿方：Flutter抱Google，Rax抱Alibaba;
2. 易用性：Flutter完爆Rax。Rax创建后只有编码环境，运行环境还得再装Weex，而Weex又是一个神奇的东西，对Native不熟的，可能会比较捉急；
3. 通用性：Flutter目前主要用来做移动端，Web端不行。Rax则通吃，包括Web；
4. Weex和RN对比：[Weex和RN对比](https://zhuanlan.zhihu.com/p/21677103)

