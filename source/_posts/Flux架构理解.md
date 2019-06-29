title: Flux架构理解
entitle: 'flux-study'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528023412
date: 2018-06-03 18:56:52
tags:
    - javascript
keywords: JavaScript, Flux
description: 来自2018.1.22的笔记：Flux架构理解
photos:
    - /img/2018/15280235400292.jpg
---


### 1、是个什么鬼？

> 如何理解 Facebook 的 flux 应用架构？

Flux 的核心就是一个简单的约定：视图层组件不允许直接修改应用状态，只能触发 action。应用的状态必须独立出来放到 store 里面统一管理，通过侦听 action 来执行具体的状态操作。
所谓的单向数据流，就是当用户进行操作的时候，会从组件发出一个 action，这个 action 流到 store 里面，触发 store 对状态进行改动，然后 store 又触发组件基于新的状态重新渲染。
即可以看出：视图组件变得很薄，只包含了渲染逻辑和触发 action 这两个职责，即所谓 "dumb components"（愚蠢组件）

### 2、结构图

![](/img/2018/15280235400292.jpg)



