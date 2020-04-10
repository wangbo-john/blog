title: Typescript入坑篇2
entitle: 'ts-study-part2'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1546832521
date: 2019-01-07 11:42:01
tags:
    - typescript
keywords: typescript, tslang
description: TypeScript入坑笔记第二篇。这一部分记录下编码规范和一些简单的语法问题。
photos:
    - /img/2018/15459066435251.jpg
---

### 规范相关

> 题外话，今天突然意识到文章英文标题好像有点怪...`ts-study`，当时为啥不写成`ts-learning`呢，明显感觉后者读起来更爽口啊😂️

资料：

- [入坑篇1（前置操作）](https://shuoit.net/tech/ts-study-part1-1528259629.html)
- [自定义 tslint & eslint 详细规则](https://juejin.im/post/5b3859a36fb9a00e4d53fc85)
- [官方语法基础](https://www.tslang.cn/docs/handbook/basic-types.html)
- [Typescript Guidelines](https://semlinker.com/ts-intro-and-guide/)
- [TypeScript Handbook（中文版）](https://zhongsp.gitbooks.io/typescript-handbook/)
- [深入理解 TypeScript](https://jkchao.github.io/typescript-book-chinese/)
- [awesome-typescript](https://github.com/semlinker/awesome-typescript)




### 编码相关

> 由于官方及其他资料在这方面非常详细了，本文不作基础内容的赘述。这里假定读者已经学习了基础的语法，如`基础类型`、`变量声明等`......

#### 1. 新建`ts`文件

可以选择在项目的根目录建立`src`目录，然后新建`index.ts`文件

```typescript
type CallBack = (value?: string) => void;

interface Config {
    name: string
    age: number
    todo?: CallBack
}

export default class Person {

    init: Config

    constructor(init?: Config) {
        if (init) {
            this.init = init;
            this.init.todo = init.todo || ((value?: string) => {
                console.log(value || 'This person have been created...But nothing todo')
            })
        }
    }

    public todoSomething() {
        // this.init.todo();
        this.init.todo('Hello World. Welcome to learn TypeScript');
    }
}
```


#### 2. 配置编译选项`tsc`、文档生成、头文件等

```json
"scripts": {
    "tsc": "tsc",
    "dev": "npm run tsc -w",
    "types": "tsc -d --emitDeclarationOnly --allowJs false --declarationDir ./@types",
    "build": "build options...",
    "prepush": "npm run tsc && npm run types",
    "prepublish": "npm run prepush && npm run build",
    "typingsdoc": "npx typedoc --out ./typings/doc  ./src/**/*.ts  --module umd"
}
```

其中这块的配置，在[part1](https://shuoit.net/tech/ts-study-part1-1528259629.html)提到过，[包括同时生成头文件并允许js文件输入](https://shuoit.net/tech/Allow--declaration-with--allowJs-1546511333.html)

这里把新增加的，简要说明下：

- dev: 监听.ts文件改动，实时编译。即增加`-w`参数可避免手动编译
- prepush: 合并了两个命令，并且`git push`时，会被触发。以此可强制推送前编译最新代码
- prepublish: 同样合并两个命令，在即将发布上线时使用
- typingsdoc: 生成Api文档。关于这个，不清楚的读者请看这篇文章：[Typedoc使用笔记](https://shuoit.net/tech/typedoc-notes-1544152560.html)

#### 3. tslint及tsconfig

```json
// tslint.json
{
  "defaultSeverity": "error",
  "extends": [
    "tslint:recommended",
    "tslint-config-prettier"
  ],
  "rules": {
    "encoding": true,
    "no-console": false,
    "object-literal-sort-keys": false,
    "interface-name": [true, "never-prefix"],
    "no-unused-expression": [true, "allow-fast-null-checks"],
    "only-arrow-functions": false,
    "no-duplicate-imports": true,
    "no-mergeable-namespace": true,
    "import-spacing": true,
    "interface-over-type-literal": true,
    "new-parens": true,
    "no-shadowed-variable": [
      true,
      {
        "class": true,
        "enum": true,
        "function": true,
        "interface": false,
        "namespace": true,
        "typeAlias": false,
        "typeParameter": false
      }
    ],
    "variable-name": false
  }
}

// tsconfig.json
{
  "compilerOptions": {
    "charset": "utf8",
    "sourceMap": true,
    "allowSyntheticDefaultImports": true,
    "target": "es5",
    "moduleResolution": "node",
    "module": "umd",
    "outDir": "./built",
    "experimentalDecorators": true,
    "removeComments": true,
    "preserveConstEnums": true,
    "allowJs": true
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "built",
  ]
}
```

#### 4. 测试编译

执行`npm run prepush`和`npm run typingsdoc`后。根目录下，应该多了俩目录：

![](/img/2019/15469365443647.jpg)

Api文档目录

![](/img/2019/15469365901446.jpg)


如果遵照第一篇的配置，加过`prettier`大法。会发现`src`下的源文件也被美美的调整了代码风格

目前，暂时就这些。后边在整理，总结！！！

