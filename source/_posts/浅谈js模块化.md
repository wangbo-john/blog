title: 浅谈js模块化
entitle: 'javascript-module-notes'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1544172600
date: 2018-12-07 16:50:00
tags: 
    - JavaScript
keywords:
description: 再来总结下JavaScript模块化的问题，虽说没什么用，但要往编译层扒，这玩意也是绕不开的。
photos:
---

关于JS模块化的问题，一直被忽视，今天重来拿出来谈谈：`CommonJS`，`AMD`，`UMD`，以及`Harmony`等...

> 没有模块化之前的操作：**script标签引入js文件，相互罗列，但是被依赖的放在前面，否则使用就会报错**。命名空间是由单一的全局对象来描述的，如`JQuery`的`$`。


#### CommonJS

主要用于NodeJS后端，采用同步的方式加载文件，它有四个重要的环境变量为模块化的实现提供支持：module、exports、require、global。实际使用时，用module.exports定义当前模块对外输出的接口（不推荐直接用exports），用require加载模块。

特点：

1. 模块可以多次加载，但是只会在第一次加载时运行一次，然后运行结果就被缓存了，以后再加载，就直接读取缓存结果。要想让模块再次运行，必须清除缓存。
2. 模块加载会阻塞接下来代码的执行，需要等到模块加载完成才能继续执行——同步加载。


#### AMD

在服务端，模块文件都存在本地磁盘，读取非常快，所以这样做不会有问题。但是在浏览器端，限于网络原因，**CommonJS不适合浏览器端模块加载，更合理的方案是使用异步加载**，即：**ADM：异步模块定义，Asynchronous Module Definition**。

AMD规范采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。

一点优点：适合在浏览器环境中异步加载模块、并行加载多个模块；
一点缺点：不能按需加载、开发成本大。

#### CMD

CMD是在AMD基础上改进的一种规范，和AMD不同在于对依赖模块的执行时机处理不同，CMD是就近依赖，而AMD是前置依赖。此规范其实是在sea.js推广过程中产生的。

```javascript

/** AMD写法 **/
define(["a", "b", "c", "d", "e", "f"], function(a, b, c, d, e, f) { 
     // 等于在最前面声明并初始化了要用到的所有模块
    a.doSomething();
    if (false) {
        // 即便没用到某个模块 b，但 b 还是提前执行了
        b.doSomething()
    } 
});

/** CMD写法 **/
define(function(require, exports, module) {
    var a = require('./a'); //在需要时申明，很明显CMD可以做到按需加载
    a.doSomething();
    if (false) {
        var b = require('./b');
        b.doSomething();
    }
});

```

#### UMD

**通用模块定义（UMD，Universal Module Definition）**。兼容AMD和commonJS规范的同时，还兼容全局引用的方式。可运行在浏览器或服务器环境。

无导入导出规范，实现原理如下：

1. 先判断是否支持Node.js模块格式（exports是否存在），存在则使用Node.js模块格式。
2. 再判断是否支持AMD（define是否存在），存在则使用AMD方式加载模块。
3. 前两个都不存在，则将模块公开到全局（window或global）。

```javascript

(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        //AMD
        define(['jquery'], factory);
    } else if (typeof exports === 'object') {
        //Node, CommonJS之类的
        module.exports = factory(require('jquery'));
    } else {
        //浏览器全局变量(root 即 window)
        root.returnExports = factory(root.jQuery);
    }
}(this, function ($) {
    //方法
    function myFunc(){};
    //暴露公共方法
    return myFunc;
}));

```

#### ES6 Module

旨在成为浏览器和服务器通用的模块解决方案。

其模块功能主要由两个命令构成：export和import。export命令用于规定模块的对外接口，import命令用于输入其他模块提供的功能。import命令会被 JavaScript 引擎静态分析，在编译时就引入模块代码，而不是在代码运行时加载，所以无法实现条件加载。也正因为这个，使得静态分析成为可能。

ES6模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块。

特点：

1. 按需加载（编译时加载）
2. import和export命令只能在模块的顶层，不能在代码块之中（如：if语句中）,import()语句可以在代码块中实现异步动态按需动态加载

语法：

1. 导入：import {模块名A，模块名B...} from '模块路径'
2. 导出：export和export default
3. import('模块路径').then()方法

> 注意：export只支持对象形式导出，不支持值的导出，export default命令用于指定模块的默认输出，只支持值导出，但是只能指定一个，本质上它就是输出一个叫做default的变量或方法。

规范：

```javascript

/*错误的写法*/
// 写法一
export 1;

// 写法二
var m = 1;
export m;

// 写法三
if (x === 2) {
  import MyModual from './myModual';
}

/*正确的三种写法*/
// 写法一
export var m = 1;

// 写法二
var m = 1;
export {m};

// 写法三
var n = 1;
export {n as m};

// 写法四
var n = 1;
export default n;

// 写法五
if (true) {
    import('./myModule.js')
    .then(({export1, export2}) => {
      // ...·
    });
}

// 写法六
Promise.all([
  import('./module1.js'),
  import('./module2.js'),
  import('./module3.js'),
])
.then(([module1, module2, module3]) => {
   ···
});

```



#### System

systemjs是模块加载器，可以导入任何流行格式的模块（CommonJS、UMD、AMD、ES6）。它是工作在ES6模块加载polyfill 之上，它能够很好的处理和检测所使用的格式。 systemjs 也能使用插件转换es6（ 用 Babel 或者 Traceur）或者转换TypeScript 和 CoffeeScript代码。你只需要在导入你的模块之前使用 System.config({ ... }) 进行系统配置。

[Dynamic ES module loader](https://github.com/systemjs/systemjs)




#### Harmony

未来的模块，目前仍处于建设性阶段。

支持基于远程来源的模块，如：

```javascript

module cakeFactory from "http://addyosmani.com/factory/cakes.js";
cakeFactory.oven.makeCupcake( "sprinkles" );
cakeFactory.oven.makeMuffin( "large" );

```


#### 相关资料：

[https://segmentfault.com/a/1190000012419990](https://segmentfault.com/a/1190000012419990)

[http://justineo.github.io/singles/writing-modular-js/](http://justineo.github.io/singles/writing-modular-js/)

[https://juejin.im/post/5b4420e7f265da0f4b7a7b27#comment](https://juejin.im/post/5b4420e7f265da0f4b7a7b27#comment)

[https://juejin.im/post/5aaa37c8f265da23945f365c](https://juejin.im/post/5aaa37c8f265da23945f365c)

[http://wiki.jikexueyuan.com/project/javascript-design-patterns/es-harmony.html](http://wiki.jikexueyuan.com/project/javascript-design-patterns/es-harmony.html)





