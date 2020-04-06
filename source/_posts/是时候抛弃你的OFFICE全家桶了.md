title: 是时候抛弃你的OFFICE全家桶了
entitle: nodeppt
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1504692052
date: 2017-09-06 18:00:52
tags:
	- 黑科技
	- javascript
keywords: nodeppt,ppt
description: 今天来说一个三年前的项目，于我而言算是是黑科技，给小伙伴们分享下使用心得。同时友情提示你，是时候抛弃office全家桶套餐了，这个更有营养。
photos:
	- /img/2017/blackTech.png
---

![blackTech](/img/2017/blackTech.png)

### 今天要说的是一个很炫的东西，叫做**NodePPT**

你是否有这种场景，辛苦做好一份`PPT`，然后发文件给其他人，奈何人家用的`macOS`且没有装微软全家桶，如果有`Keynote`还算好。想表达的意思就是通过这样的`Native`端办公软件，在跨平台协作上，难免会「丢真」。而今天的主角，将以极客化的方式为你继续`PPT`之路，让你越来越喜欢装逼......

#### 项目地址

> let's look look...  [demo](http://js8.in/nodeppt/)

源码：[nodeppt](https://github.com/ksky521/nodeppt)

文档：[推荐nodeppt：使用markdown语法来写网页ppt](http://js8.in/2013/11/16/%E6%8E%A8%E8%8D%90nodeppt%EF%BC%9A%E4%BD%BF%E7%94%A8markdown%E8%AF%AD%E6%B3%95%E6%9D%A5%E5%86%99%E7%BD%91%E9%A1%B5ppt/)

#### 安装使用啥的，官方文档说的很清楚，以下为我使用笔记

1 升级版本：

> npm update -g nodeppt

2 创建一个文档：

```
// a. 执行如下命令
nodeppt create hello

// b. 交互式信息补充
please input：
title (slide title) Hello
subtitle world
speaker (speaker) Thomas Tang
Success：hello.md, please write your slide content

// c. 使用MWeb之类的MD工具开始愉快的编写ppt吧~
```

3 启动预览

```shell
// 其中，-w表示watch模式，即：改动会时时生效，无需手动刷新浏览器。有没有很*的样子
nodeppt start -w -p 9090
```

#### 将写好的PPT作为gitpages服务发布

1 导出全部，包括nodeppt的js、img和css文件夹到执行目录下，如：docs

```
nodeppt generate . docs -a
```

> 目前发现两个问题，首先生成docs目录里，还有一个docs目录，在接着执行命令，还会继续生成。直接删掉不需要的目录；其次第一步生成的docs目录用了`Git`初始化，这就是说如果你的根目录已经用了Git管理，再套一层会导致推送到GitHub对应的目录没有文件，而`git status`会出现：**modified:   docs (modified content, untracked content)**。解决办法还是删掉`.git`和`.gitignore`


2 有了docs目录后，本地打开里面的`index.html`，看看样式对不对，没问题之后再进行第三步

3 在GitHub Pages的`Source`处，选择：`Use only the /docs folder for GitHub Pages`

4 访问配置好的域名，如`http://ppt.shuoit.net`，就可以远程访问ppt了，这并不需要你在任何平台安装office全家桶

5 剩下的工作就是发挥创造力，码字，做交互。每次写完重复以上步骤就可以使得ppt更新





