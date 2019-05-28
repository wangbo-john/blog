title: npm安装完后的回调处理
entitle: 'npm-script-postinstall'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528025660
date: 2018-06-03 19:34:20
tags:
    - JavaScript
    - 码常规
keywords: JavaScript, npm, nodejs
description: 来自2018.1.19的笔记：npm安装完后的回调处理（scripts的postinstall）
photos:
    - /img/2018/15280257721155.jpg
---

### 比如在ReactNative中，安装完依赖之后，需要改端口、删掉某些文件....

> 多个操作不能在package.json中定义数组，但可以重新定义一个脚本，在脚本中定义操作集合：

![](/img/2018/15280257721155.jpg)

shell中这么写就行

```javascript
#!/bin/bash

echo 'Now run custom commands after all package is installed.'

echo 'modify react-native package-manager default port'
react-native-port-patcher --new-port 9090

echo 'fix `Font Awesome` could not be found within the package etc.'
rm ./node_modules/react-native/local-cli/core/__fixtures__/files/package.json
```


