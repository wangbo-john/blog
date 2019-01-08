title: Typescript入坑篇1
entitle: 'ts-study-part1'
author: 唐先森
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528259629
date: 2018-06-06 12:33:49
tags:
- TypeScript
keywords: typescript, tslang
description: TypeScript入坑笔记第一篇。从没有到Hello World。除此之外，还将记录使用ESLint+Prettier来统一前端代码风格。
photos:
    - /img/2018/15459066435251.jpg
---

![](/img/2018/15459066435251.jpg)


## 以上是来自官方的中文定义（找了半天Logo，发现并没有...俩大写字母太难看）

### 1. 前置工作

在开始构建TS项目前，建议先把编码环境调整ok，即：**统一代码规范及结构，保证提交的是高质量的、同时尽量避免低级错误**，最起码看起来爽，让别人知道你不是一个随便的人...

> 接下来，记录一下**prettier**、**eslint|tslint**、**husky**、**lint-staged**套装的使用。

#### 依赖情况

```javascript
// .eslintrc.json内
{
	"extends": [
		"standard",
		"plugin:prettier/recommended"
	],
	"parserOptions": {
		"ecmaVersion": 7,
		"sourceType": "module",
		"ecmaFeatures": {
			"jsx": true
		}
	},
	"parser": "babel-eslint",
	"env": {
		"es6": true,
		"browser": true,
		"node": true
	},
	"plugins": ["react", "jsx-a11y", "import"],
	"rules": {
		"prettier/prettier": "error",
		"class-methods-use-this": 0,
		"import/no-named-as-default": 0,
		"react/jsx-filename-extension": [
			"error",
			{
				"extensions": [".js", ".jsx"]
			}
		]
	}
}

// .prettierrc内（注意这个文件没有后缀。据说是加后缀VSCode不认，本人偏爱JB全家桶，所以没试过...）
{
  "printWidth": 180,
  "tabWidth": 4,
  "useTabs": false,
  "semi": true,
  "singleQuote": true,
  "jsxSingleQuote": false,
  "trailingComma": "all",
  "bracketSpacing": false,
  "jsxBracketSameLine": true,
  "arrowParens": "always"
}

// package.json内
{
    "devDependencies": {
        "typescript": "^3.2.2",
        "typedoc": "^0.13.0",
        "babel-eslint": "^10.0.1",
        "eslint": "^5.11.1",
        "eslint-config-prettier": "^3.3.0",
        "eslint-config-standard": "^12.0.0",
        "eslint-plugin-import": "^2.14.0",
        "eslint-plugin-jsx-a11y": "^6.1.2",
        "eslint-plugin-standard": "^4.0.0",
        "eslint-plugin-react": "^7.12.0",
        "eslint-plugin-node": "^8.0.0",
        "eslint-plugin-prettier": "^3.0.1",
        "eslint-plugin-promise": "^4.0.1",
        "prettier": "^1.15.3",
        "husky": "^1.3.1",
        "lint-staged": "^8.1.0"
    },
    "husky": {
        "hooks": {
            "pre-commit": "lint-staged"
        }
    },
    "lint-staged": {
        "src/**/*.{jsx,txs,ts,js,json,css,md}": [
            "prettier --write",
            "eslint --fix",
            "git add"
        ]
    }
}
```

以上环境配好后，可以试着在`src`下建一个`*.js`文件。随便写点东西，然后咱们提交代码，看看发生了啥~

![](/img/2018/15460683493736.jpg)


嗷嗷，如此犀利是不是。以上是针对ES的，来看下TS大法的~

- 新增俩文件：**tslint.json**和**tsconfig.json**，前者是规范约束文件，后者是ts编译参数配置

```javascript
// tslint.json
{
    "defaultSeverity": "error",
    "extends": [
        "tslint:recommended",
        "tslint-config-prettier"
    ]
}

// tsconfig.json
{
  "compilerOptions": {
    "sourceMap": true,
    "allowSyntheticDefaultImports": true,
    "declaration": false,
    "target": "es6",
    "moduleResolution": "node",
    "allowJs": true,
    "module": "umd",
    "outDir": "./dist",
    "experimentalDecorators": true
  },
  "exclude": [
    "node_modules"
  ]
}

// package.json
{
    "tslint": "^5.12.0",
    "tslint-config-prettier": "^1.17.0"
}

// 其中scripts内需要配上tsc
"scripts": {
    "build": "tsc",
    "prepare": "npm run build"
}
```

关于这个架手架的源码，可以参考这里：[TypeScript](https://github.com/tangkunyin/hello-fe/tree/master/TypeScript)


到现在为止，好像关于写TS的事情并没有记录。做了太多前置工作，内容放到[part2](https://shuoit.net/tech-notes/ts-study-part2-1546832521.html)吧...

另外，叨叨一句，**EditorConfig**可以不用了（要了也是多余）

> 参考资料：

[使用ESLint+Prettier来统一前端代码风格](https://juejin.im/post/5b27a326e51d45588a7dac57)

[专治代码洁癖系列](https://juejin.im/post/5a791d566fb9a0634853400e)

