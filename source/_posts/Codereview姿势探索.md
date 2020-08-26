title: Codereview姿势探索
entitle: 'codereview-explore'
author: 托码思
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1598433342
date: 2020-08-26 17:15:42
tags: 
- 码常规
- 生产力
keywords: 代码审查，codereview，Phabricator
description: Phabricator在macOS下的姿势探索
photos:
---

### Code Review的意义

code review是现代软件开发非常有意义的一种技术管理方法，其价值不止在于代码的准入和找bug，也是所有人技术交流和成长的一种手段，总结CR的意义，不限于：

- 找出可能的坑
- 帮助新人成长
- 互相督促代码质量
- 互相学习设计理念及编码思路

理想情况下所有代码合并都必须进行code review，并且必须CC整个组，让你的每一个合并都更认真地对待

### phabricator

诞生于Facebook内部，是一套基于Web的软件开发协作工具，包括代码审查工具Differential，资源库浏览器Diffusion，变更监测工具Herald，Bug跟踪工具Maniphest和维基工具Phriction。Phabricator可与Git、Mercurial和Subversion集成使用。

其本身是PHP开源程序：[https://github.com/phacility/phabricator](https://github.com/phacility/phabricator)

#### 系统配置

PHP环境请自行google，这里记录nginx配置

```nginx
server {
    
    set $content "~/dev-lib/phabricator/webroot";

    listen 80;

    server_name phabricator.tangkunyin.com;

    root  $content;
   
    location / {
        index  index.php index.html index.htm;
        rewrite ^/(.*)$ /index.php?__path__=/$1 last;
    }

    location ~ \.php$ {
        fastcgi_pass                127.0.0.1:9000;
        fastcgi_index               index.php;
        fastcgi_intercept_errors    on;
        include /usr/local/etc/nginx/fastcgi.conf;
    }    

}
```

运行后，会提示通过命令设置**Mysql**信息，一般情况下直接设置密码即可

![](/img/2020/15984343118360.jpg)

接着，会让你更新数据库信息，同样按提示操作即可

![](/img/2020/15984345189850.jpg)
![](/img/2020/15984344260057.jpg)

如此**phabricator**就算安装完成了，接下来根据引导配置一下系统各项参数。以下这些看情况配置就好

![](/img/2020/15984364325138.jpg)


#### 项目配置

先下载并配置环境变量：https://github.com/phacility/arcanist
再设置帐号信任：

```
arc set-config default http://phabricator.tangkunyin.com/
arc install-certificate http://phabricator.tangkunyin.com/
```

执行后会提示到网站上找到对应的token，找到后粘贴输入。注意提示的网址应该是：[http://phabricator.tangkunyin.com/conduit/login/](http://phabricator.tangkunyin.com/conduit/login/)，如果不是重新登录以加载新的配置

最后在项目中创建.arcconfig文件：

```
{
  "project_id" : "your-project",
  "conduit_uri" : "https://phabricator.tangkunyin.com/"
}
```

##### review步骤：

1、git commit你的修改
2、开始review

```
arc diff commit_id --create

或修改

arc diff commit_id --update D12306  （这儿的D12306是上一次review时的revisionId）
```

3、填写review信息
![](/img/2020/15984431302261.jpg)


4、现在你会收到邮件，也可以在网站上看到你的review，请大家评审吧

5、merge仍然要在gitlab中完成，merge消息中要带上phabricator的review地址，证明这次merge是review过的，收到了accept的review才进行合并


**需要注意的是我这里并没有提到邮件发送配置，实际过程是要配置的，不然上边的邮件肯定发不出去**

具体参考这篇文章：https://cloud.tencent.com/developer/article/1609447


