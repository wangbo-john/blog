title: typedoc使用笔记
entitle: 'typedoc-notes'
author: 唐先森
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
date: 2018-12-07 11:16:52
timestamp: 1544152560
tags: 
    - TypeScript
    - 开发日常
keywords: TypeDoc,TypeScript
description: TypeDoc 是一款支持 TypeScript 的文档生成工具，安装、使用方便。最后生成的是静态的 HTML 文件，界面简洁，提供多个可选的配置，并且可以按照自己的需求自定义界面样式。
photos:
---

**TypeDoc 是一款支持 TypeScript 的文档生成工具。**

安装、使用方便。最后生成的是静态的 HTML 文件，界面简洁。

提供多个可选的配置，并且可以按照自己的需求自定义界面样式。

源码地址：[TypeDoc](https://github.com/TypeStrong/typedoc)

文档地址：[TypeDoc Documentation](https://typedoc.org/api/index.html)

![](/img/2018/15445225439852.jpg)


### 优雅的使用方式

```javascript

// 1. 配置到package.json中，其中使用npx命令，无需单独安装typedoc库
// 2. transform.js里可以将目标文档的特殊关键字替换。做一些前置处理

"scripts": {
    "typingsdoc": "node typings/transform.js && npx typedoc --out ./typings/doc  ./typings  --module umd"
},

```


### 两个好用的插件

[typedoc-plugin-single-line-tags](https://github.com/christopherthielen/typedoc-plugin-single-line-tags)

以上插件可以用一个注解，将描述展示为一行。而非多行，比如一个私有类，可以增加一个@private，然后增加一段说明


[typedoc-plugin-external-module-name](https://github.com/christopherthielen/typedoc-plugin-external-module-name)

当有多个模块，分为不同文件时，可以用上面的插件将模块区分出来。做类似的二级栏目。


### 关于同一Module文档的二级分类

1. 使用上面的插件，按文件区分；
2. 使用这个插件搞定：[typedoc-plugin-toc-group](https://github.com/tangkunyin/typedoc-plugin-toc-group)


> 至于插件的开发，请移步这篇文章：


[TypeDoc插件开发小记](https://shuoit.net/tech-notes/how-to-create-plugin-for-typedoc-1545808023.html)


