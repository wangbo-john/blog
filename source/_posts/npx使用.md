title: npx使用
entitle: 'what-is-the-npx'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
date: 2018-12-07 11:36:34
timestamp: 1544153760
tags: 
    - 码常规
keywords: npx
description: npx命令说明，及用法简介。
photos:
---

`npx`是`npm v5.2.0`引入的一条命令，引入这个命令的目的是为了提升开发者使用包内提供的命令行工具的体验。


举例：使用**create-react-app**创建一个react项目。

老方法：
```
npm install -g create-react-app
create-react-app my-app
```

npx方式：
```
npx create-react-app my-app
```

这条命令会临时安装 create-react-app 包，命令完成后create-react-app 会删掉，不会出现在 global 中。下次再执行，还是会重新临时安装。

也就是说 npx 会自动查找当前依赖包中的可执行文件，如果找不到，就会去 PATH 里找。如果依然找不到，就会帮你安装！

npx 甚至支持运行远程仓库的可执行文件：
```
npx github:piuccio/cowsay hello
```

主要特点：

1. 临时安装可执行依赖包，不用全局安装，不用担心长期的污染。
2. 可以执行依赖包中的命令，安装完成自动运行。
3. 自动加载node_modules中依赖包，不用指定$PATH。
4. 可以指定node版本、命令的版本，解决了不同项目使用不同版本的命令的问题。


