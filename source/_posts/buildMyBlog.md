---
title: 使用Hexo + GitHub Pages 创建我的博客网页
reward: true
copyright: false
date: 2020-03-20 19:46:30
tags: 
	- 博客创建
	- Hexo
	- Git
categories: 
   - 学习
   - 博客
---

# 准备阶段

1. 准备[nodejs](https://nodejs.org/en/) 需要用到npm；

2. 准备好[git](https://git-scm.com/)终端

3. 创建自己的[GitHub](https://github.com/)账号，并新建一个仓库，名为：`username.github.io`，`username`就是你的GitHub账号名称

4. 在本地端使用`git bash here`生成一个key：

    ~~~sh
    ssh-keygen -t rsa -C 你注册GitHub时的邮箱
    ~~~

5. 然后去` c/Users/you/.ssh/id_rsa.pub` 路径文件打开文件，复制里面的内容到GitHub里个人主页的Setting下面，选择`SSH and GPG keys` ,接着添加key，把刚刚复制的内容填进去；![170ba659dab859](https://gitee.com/wycisme/imageBed/raw/master/img/170ba659dab859.jpg)

6. 接着初始化自己的github信息

    ~~~ sh
    git config --global user.name 注册github时的用户名
    git config --global user.email 注册github时的邮箱
    ~~~

7. 打开`git bash here`，执行`ssh -T git@github.com`，之后会出现一系列的yes or no的问题，我们只需要回答yes即可，最终会输出如下类似内容：

   ~~~
   Hi username! You've successfully authenticated
   ~~~

   说明本机已经可以连接到自己的GitHub了

# 安装Hexo

1. 在做完准备工作后，就可以安装hexo了,找个文件夹，单击鼠标右键选择git bash here ：

   ~~~ sh
   npm install hexo-cli -g
   npm install hexo --save
   ~~~

2. 初始化hexo：

   ~~~ sh
   hexo init 本地存放目录（如：myBlog）
   cd myBlog
   ~~~

3. 自动安装网站所需组件：

   ~~~ sh
   npm install
   ~~~



## Hexo 主题选择

主题官网为：<https://hexo.io/themes/> 

我这里选择的是Ayer <https://shen-yu.gitee.io/2019/ayer/> 

![hexo-theme-ayer](https://gitee.com/wycisme/imageBed/raw/master/img/5e739724e83c3a1e3ab53941.jpg)

桌面，手机端都有适配，而且很好看

* 安装主题，在本地的博客根目录下：

  ~~~sh
  git clone https://github.com/Shen-Yu/hexo-theme-ayer.git themes/ayer
  ~~~

  

* 将博客根目录下的 `_config.yml` 里的 `theme` 值修改成 `ayer`

  ~~~json
  theme: ayer
  ~~~

* 关于主题[配置](https://shen-yu.gitee.io/2019/ayer/#%E4%B8%BB%E9%A2%98%E9%85%8D%E7%BD%AE)

> 参考：<https://www.cnblogs.com/LiT-26647879-510087153/p/12433023.html>