title: WebStorm语法高亮问题
entitle: 'highlight-webstorm'
author: 唐先森
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528025404
date: 2018-06-03 19:30:04
tags:
    - WebStrom
keywords: WebStrom, highlight-webstorm
description: 来自2018.1.23的笔记：WebStorm语法高亮问题。一定程度上解决React-Native开发过程中，WebStorm语法高亮的问题
photos:
---

## 一定程度上解决React-Native开发过程中，WebStorm语法高亮的问题

###  Cannot resolve symbol ‘Component’ & Cannot resolve symbol‘PropTypes’

```
1.解决 Cannot resolve symbol 'Component' 

  安装依赖：npm install @types/react --save
  调用方法：import React, { Component } from 'react'

2.解决 Cannot resolve symbol 'PropTypes'

  安装依赖：npm install prop-types --save
  调用方法：import PropTypes from 'prop-types'
```

https://www.npmjs.com/package/@types/react-native

> npm install --save @types/react-native

