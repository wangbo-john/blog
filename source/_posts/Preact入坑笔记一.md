title: Preact入坑笔记一
entitle: 'Preact-learning-notes1'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1548753818
date: 2019-01-29 17:23:38
tags:
    - 前端
    - React
    - TypeScript
keywords: Preact，Preact学习，Preact和TS混用
description: Preact，一个体积只有3kB的React替代版本。其简单易用，据资料显示性能甩React和Vue好几条街，于是便有了以本文为开始的探索。
photos:
    - /img/2019/15487547031177.jpg
---

### 简述

**Preact**，读：`['pri:ækt]`，而非`批React`。跟Pure React也扯不上什么关系。

是由谷歌的一~~群~~大兄弟开发和维护的优化版`React`。这并非造个轮子要取代FB的`React`，通过一些资料和实践，发现这个轮子并不简单。具体可以链接到官网，中文说明足够友好，作者真是太赞了。

官方：[Preact官网](https://preactjs.com/)
项目地址：[Preact](https://github.com/developit/preact)

![](/img/2019/15487547031177.jpg)


### 实践笔记，第一部分

这也并非是近期的新鲜事物，只是我个人刚接触罢了。从这一个月的使用看来，这个确实比React简单一些。比如核心库直接去掉了`PropTypes`这种东西，对`Render`也进行了优化，搭配`TypeScript`和`Less`使用，爽的么法...

#### 一. 这里记录下开发环境的配置，主要依赖情况：

- Preact: ^8.2.6
- TypeScript: ^3.2.4
- Less: ^3.9.0
- Webpack: ^4.29.0

```json
"scripts": {
        "dev": "NODE_ENV=development webpack-dev-server --config scripts/webpack.config.js",
        "build": "NODE_ENV=production webpack --config scripts/webpack.config.js",
        "ts-build": "tsc --pretty --outDir ./modules",
        "ts-watch": "tsc -w --pretty --outDir ./modules",
        "pm2-dev": "pm2 start npm --name prdemo -- run dev && npm run ts-watch",
        "pm2-clear": "pm2 delete prdemo"
    },
    "husky": {
        "hooks": {
            "pre-commit": "lint-staged"
        }
    },
    "lint-staged": {
        "linters": {
            "*.{js,jsx,md,json}": [
                "prettier --write",
                "git add"
            ],
            "*.{ts,tsx}": [
                "prettier --write",
                "tslint --fix",
                "git add"
            ],
            "*.css": [
                "stylelint --fix",
                "git add"
            ],
            "*.less": [
                "stylelint --fix --syntax=less",
                "git add"
            ]
        },
        "ignore": [
            "./modules/**"
        ]
    },
    "eslintConfig": {
        "extends": "eslint-config-aerian"
    },
    "eslintIgnore": [
        "build/*"
    ],
    "stylelint": {
        "extends": "stylelint-config-standard",
        "rules": {
            "indentation": 4,
            "number-leading-zero": null,
            "at-rule-no-unknown": [
                true,
                {
                    "ignoreAtRules": [
                        "plugin"
                    ]
                }
            ],
            "selector-pseudo-class-no-unknown": [
                true,
                {
                    "ignorePseudoClasses": [
                        "global"
                    ]
                }
            ]
        }
    },
    "dependencies": {
        "preact": "^8.2.6"
    },
    "devDependencies": {
        "@types/jest": "^23.3.13",
        "@types/node": "^8.10.38",
        "copy-webpack-plugin": "^5.0.0",
        "css-loader": "^2.1.0",
        "file-loader": "^3.0.1",
        "html-webpack-plugin": "^3.2.0",
        "husky": "^1.3.1",
        "jest": "^24.0.0",
        "less": "^3.9.0",
        "less-loader": "^4.1.0",
        "lint-staged": "^8.1.1",
        "mini-css-extract-plugin": "^0.5.0",
        "mkdirp": "^0.5.1",
        "preact-async-route": "^2.2.1",
        "preact-render-spy": "^1.3.0",
        "preact-router": "^2.6.1",
        "prettier": "^1.16.1",
        "recursive-copy": "^2.0.10",
        "style-loader": "^0.23.1",
        "stylelint": "^9.10.1",
        "stylelint-config-standard": "^18.2.0",
        "ts-jest": "^23.10.5",
        "ts-loader": "^5.3.3",
        "tslint": "^5.12.1",
        "tslint-config-prettier": "^1.17.0",
        "tslint-consistent-codestyle": "^1.15.0",
        "tslint-eslint-rules": "^5.4.0",
        "tslint-react": "^3.6.0",
        "typescript": "^3.2.4",
        "typings-for-css-modules-loader": "^1.7.0",
        "uglifyjs-webpack-plugin": "^2.1.1",
        "url-loader": "^1.1.2",
        "webpack": "^4.29.0",
        "webpack-cli": "^3.2.1",
        "webpack-dev-server": "^3.1.14"
    }
```

之所以加了`pm2`，是因为有两份`TypeScript`源码需要watch，`Webpack`本身watch占用一个console窗口，tsc自己也要有。如果一个命令启动，就会导致一个卡在另一个之前，导致另一个无法启动.....所以索性把webpack直接放到了后台。其本身作为Server服务，也算合理。不过正常情况，不会像我这么变态......

#### 二. TSC

```json
{
    "compilerOptions": {
        "charset": "utf8",
        "sourceMap": true,
        "rootDir": "./src/",
        "outDir": "./modules",
        "skipLibCheck": true,
        "allowSyntheticDefaultImports": true,
        "target": "es5",
        "module": "es6",
        "lib": ["es6", "es7", "dom"],
        "allowJs": false,
        "jsx": "react",
        "jsxFactory": "h",
        "strict": true,
        "declaration": true,
        "moduleResolution": "node",
        "noImplicitAny": false,
        "esModuleInterop": false,
        "experimentalDecorators": true,
        "removeComments": false,
        "preserveConstEnums": true
    },
    "include": ["./src/**/*"],
    "exclude": [
        "docs",
        "modules",
        "typings",
        "node_modules"
    ]
}
```

注意`jsxFactory`使用的是`h`，这个`h`是`Preact`的核心之一

#### 三. Less的一点问题

1，正常使用import引用less时，IDE会报错提示`Cannot find module './style.less'.`。直接加`@ts-ignore`可以忽略，但不是好方法

```
// @ts-ignore
import style from './style.less';
```

如上所以，解决方式也很简单：第一新建`less.d.ts`文件，第二在该文件内申明`less`

```
declare module "*.less";
```

然后，在基类中引用，注意不是import，而是**reference**指令

```
/// <reference path="../../typings/less.d.ts" />
```

2，组件上，用点语法使用less的选择器。但又得注意，`style.className`的值是随机字符串，并非对象，因此不能接着点（内层嵌套拿不到值），即：`style.className.Name1`非法

```less
a {
    &.normal {
        color: #333;
    }
    &.active {
        color: #fff;
    }
}
```

以上用法如下：

```html
<nav class={isActive ? style.active : style.normal}>{txt}</a>
```

#### 四. Preact无状态组件使用

```typescript
import {FunctionalComponent, h} from 'preact';
import style from './style.less';

interface INavProps {
    transparent?: boolean,
    title?: string,
    onBack?: (index) => void
    onHome?: (index) => void,
    onShare?: (index) => void
}
export const Navigator: FunctionalComponent<INavProps> = ({transparent, title, onBack, onHome, onShare}) => {
    const onBackPress = (e) => {
        if (onBack) {
            onBack(e);
            return;
        }
        // TODO.
    };
    const onHomePress = (e) => {
        if (onHome) {
            onHome(e);
            return;
        }
        // TODO.
    };
    const onSharePress = (e) => {
        if (onShare) {
            onShare(e);
            return;
        }
        // TODO.
    };
    const className = transparent ? style.transparent : style.normal;
    return (
        <nav class={className}>
            <button class={className} onClick={onBackPress} />
            <span class={className}>{title}</span>
            <button class={className} onClick={onHomePress} />
            <button class={className} onClick={onSharePress} />
        </nav>
    );
};
```

#### 五. 其他方面

由于着手的这项目有点怪，所以自制了一款webpack插件，其中有用到了`recursive-copy`。需要把指定文件夹里的非TS和TSX拷贝到另外的目录，配置其`filter`时曾让人蛋疼不已.....具体请看[minimatch](https://github.com/isaacs/minimatch#usage)。结论是这玩意跟一般正则不一样，当时按正则咋配咋不对

```
// 过滤ts和tsx
const OPTIONS = {
    overwrite: true,
    expand: true,
    dot: true,
    filter: ['**/*.!(tsx|ts)'],
};
```

### 扩展阅读：

- [Preact：一个备胎的自我修养](https://juejin.im/post/5a0191f25188254de1699b0b)
- [TS-Preact](https://dominicstpierre.com/how-to-start-with-typescript-and-preact-a9ea3e0ba4dc)
- [preact-material-components](https://github.com/prateekbh/preact-material-components)

