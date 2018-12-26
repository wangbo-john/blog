title: TypeDoc插件开发小记
entitle: 'how-to-create-plugin-for-typedoc'
author: 唐先森
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1545808023
date: 2018-12-26 15:07:03
tags: TypeScript
keywords: TypeDoc,TypeScript
description: 如何为TypeDoc文档系统开发插件？本文记录了typedoc-plugin-toc-group的开发流程。希望对有需求的朋友一定帮助。
photos:
---

### 关于TypeDoc是个什么鬼及其使用，可以阅读之前那篇文章

[typedoc使用笔记](https://shuoit.net/tech-notes/typedoc-notes-1544152560.html)

## 以下记录TOC二级菜单插件的开发

先明确你自己的需求，你要做什么？现有的插件有没有能满足你的，如果有类似的，改改是否能用？如果毛都没有，那就从头写吧！

造轮子有一定代价，官网如果没有良好的教程指导，那将会很令人神伤。而TypeDoc就是这样，至少我谷歌了好久，没找到类似教程的东西，索性一把梭的去看了好多插件的源码实现。还好明白套路...

插件的作用无外乎是对原有的系统做了些许扩展。其原理在于某个时刻做某些拦截，然后构造必要的数据，然后再根据主题模板进行文件输出渲染。比如我们要改造TOC模块，使其支持二级栏目：

### 1. GitHub找到TypeDoc源码，找到TOC模块所在

[TocPlugin.ts](https://github.com/TypeStrong/typedoc/blob/master/src/lib/output/plugins/TocPlugin.ts)

```

initialize() {
    this.listenTo(this.owner, {
        [PageEvent.BEGIN]: this.onRendererBeginPage
    });
}

private onRendererBeginPage(page: PageEvent) {        
    // 以上省略.... 可以看到这里，右边原有的TOC目录树就是从这里生成数据。而我们要改的可能也在这里
    TocPlugin.buildToc(model, trail, page.toc, tocRestriction);
}

```

### 2. 研究其他插件的入口文件index.js，我们可以发现

所有插件都有类似的注册方法，即：你的插件名称，功能需要挂接到文档系统。当开始生成数据时，收到事件后开始构造需要的数据结构

```

const P = require('./plugin');

module.exports = function(PluginHost) {
	const app = PluginHost.owner;
	
	// 如果已经注册过了，就别再重复注册
	if (app.converter.hasComponent(P.PLUGIN_NAME)) {
		return;
	}
	if (app.renderer.hasComponent(P.PLUGIN_NAME)) {
		return;
	}

  // 此处声明插件名称
	app.options.addDeclaration({ name: P.PLUGIN_NAME, short: P.PLUGIN_SHORT_NAME });

  // 监听数据变化
	app.converter.addComponent(P.PLUGIN_NAME, P.TocGroupPlugin);

  // 监听页面渲染
	app.renderer.addComponent(P.PLUGIN_NAME, P.TocGroupPlugin);
};

```

### 3. 做必要的监听及数据模型构造

研究TOC主题模板可以看到下图所示，其实最终读的就是toc这个变量里的children。而要改造的就是把标签内容再包一层，塞到这个变量里。弄成二维数组即可。

![](/img/2018/15458121120623.jpg)


需要注意的是，**一定要在`initialize`方法里正确的监听事件。否则拿不到你要的值**。正确监听的前提还必须是在`index.js`里正确注册。

比如要处理`PageEvent`事件，就要监听`rendered`类型；处理`Converter`事件，就要监听`converter`类型。

#### 获取文档中指定注解名称和内容

```javascript

// 1. initialize里监听： [Converter.EVENT_RESOLVE_BEGIN]: this.onBeginResolve,
// 2. onBeginResolve回调中拿到Context，根据需求取或存数据
const groupedData = [];
const deprecatedData = new Set();
const mapedTocData = {};
const reflections = context.project.reflections;
for (const key in reflections) {
    const ref = reflections[key];
    const comment = ref.comment;
    const homePath = `modules/_index_.${context.project.name.replace(/\-/g, '')}.html`;

    if (!comment || !comment.tags) continue;

    for (const tag of comment.tags) {
    // add deprecated item names
    if (DEPRECATED_REGEXP.test(`@${tag.tagName}`)) deprecatedData.add(ref.name);
    // add special tags
    if (this.regexp.test(`@${tag.tagName}`)) {
        groupedData.push(ref.name);
        const groupKey = tag.text.split(/\r\n?|\n/)[0];
        if (!mapedTocData[groupKey]) mapedTocData[groupKey] = [];
            mapedTocData[groupKey].push(ref.name);
				 break;
        }
    }
}

// 以上构造的数据需要存到Context中，试过插件的成员变量，但是在其他事件回调中拿不到成员变量的值，事件关系没有深扒....
context.project[PLUGIN_NAME] = { groupedData, deprecatedData, mapedTocData, homePath };

```

#### 构造我们所需要的数据

请直接参考这里代码：[buildGroupTocContent](https://github.com/tangkunyin/typedoc-plugin-toc-group/blob/master/plugin.ts)


### 4. 测试功能

将插件目录下的文件组织好，配置编译命令。然后拷贝到`node_modules`下，执行`typedoc`构建时，将会自动加载插件，无需手动配置：

> Loaded plugin xxx/node_modules/typedoc-plugin-toc-group

如果控制台能打印类似的这条命令，那恭喜，你的新插件已经成功加载了。然后剩下的就是反复调试。直到实现你要的功能即可。


### 5. 定制主题

默认情况下，数据结构有了，页渲染出你要的结果了。但是可能长相不太好看。这时你就要定制一下默认的主题了。思路就是猜测你的功能所在的文件，然后去改造他。比如上面截图那个。

改完了之后，你可以选择提交到npm，也可以放到自己项目中，用路径的方式去引用。

### 6. 总结一下

1. 明确需求，猜测与你功能相关的代码在哪里，去看源码；
2. 研究其他插件或主题的目录结构，搭建自己的插件或主题目录结构；
3. 入口文件注册事件，监听事件。拿到注解上的内容；
4. 当页面开始渲染时，根据注解去解析、拼凑你要的数据结构；
5. 将原来的数据结构从内部改掉（注意别改的太离谱否则加载会崩溃）；
6. 数据没问题后，定制主题。改成你喜欢的样子；

就写到这里，有不明白了。可以文章后留言，或者直接去看我写的这个主题。


