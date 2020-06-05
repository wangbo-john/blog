title: 记一次mysql再安装的坑史
entitle: 'reinstall-mysql'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1591346229
date: 2020-06-05 16:37:09
tags:
- 码常规
- 麦克
keywords: mysql，brew install mysql
description: mysql再次安装的「黑历史」，附带出坑姿势
photos:
---

### 背景

多年前帮朋友写过一个PHP的项目用过Mysql，后来硬盘空间不够了，所以卸载了。当时只是卸载没有清理干净，毕竟默认情况下卸载Mysql是不会自动删除**db**文件夹的，于是今天又有相关需求再次安装时，翻车了...

```
(HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (38)
```


#### 解决姿势

起初以为**mysql.sock**这个文件没有，另外`brew services restart mysql`再怎么启动都也不明显报错....于是[各种操作配置](https://segmentfault.com/q/1010000000094608)，一顿搞还是连不上。于是怒看Log...

![](/img/2020/15913469404177.jpg)

上图是`/usr/local/var/mysql/KunyinTang.local.err`中发现的，看上去跟我之前那个版本不兼容导致安装后服务没启动.....

于是，一顿操作卸载并删除了所有相关的老的、新的mysql文件....整理的操作如下：

```bash
cd /usr/local/etc
rm -rf my.cnf my.cnf.default


# 旧的数据文件如果有用要记得保存一份！
sudo rm -rf /usr/local/mysql
sudo rm -rf /usr/local/var/mysql

sudo rm -rf /Library/StartupItems/MySQLCOM
sudo rm -rf /Library/PreferencePanes/My*
sudo rm -rf ~/Library/PreferencePanes/My*
sudo rm -rf /Library/Receipts/mysql*
sudo rm -rf /Library/Receipts/MySQL*
sudo rm -rf /private/var/db/receipts/mysql
```

删完之后再用**brew**装一遍，问题解决了....

```bash
brew install mysql

# 看看是不是启动了
lsof -i :3306
```

![](/img/2020/15913475899004.jpg)


再然后就可以愉快的玩了.....

![](/img/2020/15913478197693.jpg)



