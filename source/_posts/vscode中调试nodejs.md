title: vscode中调试node.ts
entitle: 'debug-nodets-in-vscode'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1593399677
date: 2020-06-29 11:01:17
tags:
- typescript
- 后端
- 码常规
keywords: nodejs,typescript,debug
description: vscode中调试nodets。node：v12.16.0，框架：nest，typescript：3.7.4+，vscode：1.46.1，OS：macOS 10.15.5
photos:
---

### 背景

纯前端程序用Chrome大法调试自然不易翻车，另外打log也能输出对象、数组等复杂数据类型。但这一切换到nodejs环境就没那么爽了，黑窗口里输出复杂数据类型不仅看的眼花又缭乱，而且跟踪断点非常蛋疼，所以这就需要借助IDE能力去断点调试。

谷歌给出的文章时间上都比较老舅，所以写了这个笔录。本文采用vscode + nest搭配，系统环境在macOS下

> vscode版本是：1.46.1，开始之前最好配置一些偏好设置

![](/img/2020/15934149425950.jpg)

![](/img/2020/15934151685607.jpg)




### 调试步骤

确认启动调试的命令是否ok，主要看start命令是否有**--debug**参数

![](/img/2020/15934007141581.jpg)

点「甲壳虫」后，出现如下界面，接着点**Node.js Debug Terminal**

![](/img/2020/15934008571189.jpg)

不出意外，nodejs就可以调试了（得在js文件上打断点，如果是ts则找build后的js文件）......

**此时，如果在ts源文件中直接打断点，可以看到的是，没有反映....**，于是我们来折腾一下这块

#### 配置TypeScript调试

##### 姿势一

`tsconfig.json`中开启**sourceMap**，即：`"sourceMap": true,`。一行见效

![](/img/2020/15934142161875.jpg)


##### 姿势二

`ts-node`大法

详见资料：[使用ts-node和vsc来调试TypeScript](https://segmentfault.com/a/1190000010605261)


### 延伸阅读

- [使用 VS Code 调试 Node.js 的超简单方法](https://juejin.im/post/5cce9b976fb9a0322415aba4)
- [deno vs ts-node](https://mlog.club/article/5002614)

