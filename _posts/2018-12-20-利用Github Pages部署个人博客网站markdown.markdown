---
layout:     post
title:      "利用Github Pages部署个人博客网站"
subtitle:   "————jekyll + Hux Blog Theme + Github Pages"
date:       2018-12-20 11:00:00
author:     "LiangFan"
header-img: "img/my-post/07.jpg"
tags:
    - jekyll 
    - Github Pages 
    - 个人网站
---

> 前面已经讲过了利用Tomcat和阿里云ESC服务器来搭建个人博客网站，这个技术方案的主要优点是访问速度快，而且不用托管代码(私密性)。现在介绍另外一种部署方案——利用Github Pages。

利用Github Pages来部署个人网站的优点是代码在线上，随时可以操作，而且不用申请国内的服务器和域名，所以流程就相对简单一些，今天部署今天就能看见自己的网站并访问。


缺点嘛，一来访问速度可能会很慢特别是需要加载图片的时候，然后也会有被wall的风险，毕竟github是国外网站；二来域名会比较长不那么个性；三来需要学会操作git…当然，一和二可以通过购买域名并进行cname域名绑定和网络加速来优化，三的话本身也不是一个难题。


下面看一下简要的部署流程。


### 1.注册一个github账号
这个没啥需要讲的了，就自己百度搜下github，进入官网按照流程注册一个自己的账号。

### 2.了解并熟悉操作git
如果是首次接触可能有点难以理解，但是git说白了就是：远程线上的代码（远程仓），本地也有代码（本地仓），怎么进行本地和远程的同步以及版本管理等等。git遵循success or nothing的事务性操作命令（是不是想到了数据库，orz……），来保证ACID——原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）原则。

[关于git的环境配置和操作，可以看这里。](https://blog.csdn.net/qq_38716242/article/details/79380825)

### 3.简单了解下Github Pages
[官网入口在这里。](https://pages.github.com/)

 

### 4.在github上建立一个name.github.io的repo

例如（由于我已经建立过，所以会有红色警示）：
![Alt text](/img/my-post/20181219a.png)

### 5.将博客代码推到name.github.io这个repo
这里需要注意的是，代码都是公开的，github非付费账号的repo代码都是公开的，如果有隐私信息请注意慎重。

- 首先，将[Hux Blog Theme](https://github.com/huxpro/huxpro.github.io/)的代码clone到本地。

- 然后，将本地代码初始化后push到`name.github.io`这个repo。

little tips:
```
1）新建Text文件夹作为仓库根目录（文件夹名字随意命名）

2）将需要上传的代码文件加入到Text根目录

3）在根目录下建立仓库
使用命令行或Git Bash，输入下面命令：先进入到Text根目录下，再输入git init（初始化一个仓库）


add和commit其实相当于本地操作了~~
4）将所有文件添加到仓库
使用命令行或Git Bash，输入下面命令：git add .

5）提交
使用命令行或Git Bash，输入下面命令：git commit -m “CommitInfo”

6）添加源到GitHub
git remote add origin git@github.com:YourName/YourRepositroy.git

7）上传源到GitHub
git push -u origin master
```

- 接下来在浏览器中打开`name.github.io`就能看到自己的博客内容了。

- 后续更新博客内容可以直接在github中修改代码，或者在本地修改后和远程仓同步

 

### 6.实现keynote翻页效果

1）参考Hux在其github中的描述，了解网页实现keynote布局的情况

2）将Hux在github上的`js-module-7day`项目clone到本地

3）在自己的gith上面建一个repo，由于这里先是自己测试，可以也叫js-module-7day

4）将本地代码推到自己的github repo下，命令如：`git push origin master:gh-pages`

5）在博客的markdown文件中引用这个keynote repo的地址即可查看内容

6）通过keynote repo的代码来修改keynote内容

[实现效果可参考这里](https://huangxuan.me/js-module-7day/#/)， 还是很美观的。
















