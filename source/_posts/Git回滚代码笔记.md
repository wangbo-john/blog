title: Git回滚代码笔记
entitle: 'git-revert'
author: 唐先森
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528020667
date: 2018-06-03 18:11:07
tags:
    - git
keywords: git, git revert
description: 来自2017.3.10的笔记：Git回滚代码笔记
photos:
    - /img/2018/15280208525260.jpg
---

总有那么一次操作后，想反悔，那么git log先：

![](/img/2018/15280208525260.jpg)



此时比如想恢复红框那个版本，执行：

> git checkout 19d46ca0715df5223d9e30ba9743fc9d95a3bf78

命令结束后，会跳到HEAD分支

复制改项目到另一边，切回之前的分支，把备份覆盖回去。在 git diff 。对比一致后，在提交


