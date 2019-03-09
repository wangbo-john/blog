title: pm2控制多个ReactNative控制台
entitle: 'pm2-rn-combine'
author: 唐先森
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528026635
date: 2018-06-03 19:50:35
tags:
    - ReactNative
    - JavaScript
    - 开发日常
keywords: ReactNative, npm, nodejs
description: 来自2018.1.18的笔记：pm2控制多个ReactNative控制台。当前前提是得先改了RN默认端口。
photos:
    - /img/2018/15280267906540.jpg
---

### 一、pm2小试牛刀

#### 1、安装PM2：[pm2](https://github.com/Unitech/pm2)
> npm install pm2 -g

#### 2、常用命令：
> pm2 ls
![](/img/2018/15280267906540.jpg)

> pm2 monit

![](/img/2018/15280268329791.jpg)

#### 3、 常用的服务启动命令：
```
// 从某个文件作为服务入口启动
pm2 start app.js
​
// 启动所有定义在packge.json中的服务
pm2 start package.json
​
// 启动一个Node应用程序
pm2 start npm -- start
```

### 二、ReactNative默认端口修改

> 默人情况下，ReactNative的PackageManager端口是8081。

#### 1、临时修改端口（这个命令是2017年8月1号以后增加的功能）
> // 监听9999
> react-native start --port 9999

#### 2、永久修改

planA：
> 手动修改所有涉及端口的文件，具体文件可谷歌

planB：

> https://github.com/ktonon/react-native-port-patcher
> script中加postinstall，devDependences加

```
"postinstall": "react-native-port-patcher --new-port 9092"
"react-native-port-patcher": "^1.0.2"
```

安装以后，执行： yarn install | npm install 
打开Xcode后，重新编译，就可使用新的端口

### 三、pm2控制多个ReactNative服务

#### 启动一个名为xxName的进程，可多个一起
> pm2 start npm --name xxName -- run start
​
#### 这种方式会启动一个名为npm的进程，如果不区别名称，则另外一个无法启动
> pm2 start npm -- start


![](/img/2018/15280271298874.jpg)


