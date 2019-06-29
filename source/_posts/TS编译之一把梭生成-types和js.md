title: TS编译之一把梭生成@types和js
entitle: 'Allow--declaration-with--allowJs'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1546511333
date: 2019-01-03 18:28:53
tags: 
    - typescript
keywords: typescript,tsconfig,ts编译配置
description: 一个变通的方式，解决tsc不能同时配置declaration和allowJs都为true的姿势。本文试着采用一把梭的方式编译ts文件、并生成types
photos:
---

### 先了解下情况

如果`tsconfig.json`中，同时配置了如下操作：

```json
{
  "compilerOptions": {
    "charset": "utf8",
    "sourceMap": true,
    "allowSyntheticDefaultImports": true,
    "target": "es5",
    "moduleResolution": "node",
    "module": "umd",
    "outDir": "./dist",
    "experimentalDecorators": true,
    "removeComments": true,
    "preserveConstEnums": true,
    "diagnostics": true,
    "allowJs": true,// 允许编译javascript文件
    "declaration": true,//生成相应的 .d.ts文件（类似Objc中的.h文件）
    "declarationDir": "./@types"//单独为头文件指定存放的位置
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist"
  ]
}
```

从这里开始，allowJs如果为true，则declaration就不应该存在。否则你将会看到编译时如下的报错...

> Option 'allowJs' cannot be specified with option 'declaration'.

![](/img/2019/15465174447511.jpg)

看目录，好像需要的也都生成了！

![](/img/2019/15465177839896.jpg)

但是作为一个优秀发程序员，能眼睁睁看到编译报错而不管嘛，当然不能！！！于是去撸`github`，发现早在2016年，微软那边就知道这事：[issues-7546](https://github.com/Microsoft/TypeScript/issues/7546)，但为毛现在还存在这个问题....

于是看他们讨论~~互怼~~，无意中看到巴基斯坦一兄弟的姿势。果断试了下，哎，别说，行了....

![](/img/2019/15465180905253.jpg)


### 我把上边的操作整理了下，变成了一把梭，请看

```json
"scripts": {
    "tsc": "tsc",
    "types": "tsc -d --emitDeclarationOnly --allowJs false --declarationDir ./@types",
    "prepublish": "npm run tsc && npm run types"
},
```

于是，之后你需要执行`npm run prepublish`就可以了。前提是把下边这两句从`tsconfig.json`中干掉

```json
{
    "declaration": true,
    "declarationDir": "./@types"
}
```

如果读者有更成熟的方案，麻烦告知。感谢！




