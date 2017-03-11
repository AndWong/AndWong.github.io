---
title: Ubuntu下使用Hexo创建Gitbub博客
date: 2016-01-05 14:06:55
tags: github
---
1.环境配置
```
安装 node.js : $sudo apt install nodejs-legacy
安装 npm ： $sudo apt install npm
安装 git : $sudo apt-get install git
安装 hexo ： $sudo npm install hexo -g
```
<!--more-->
2.注册github帐号，并创建仓库（注意仓库必须以 “用户名.github.io”）

3.初始化博客，命令行输入下列命令：
```
$git init
$git remote add origin "git url"
$git pull origin master

$hexo init
$hexo s -g #预览
```

编辑根目录下的_config.yml 末尾添加下列代码:
```
  deploy:
  type: git
  repository: https://github.com/AndWong/AndWong.github.io.git
  branch: master
  ```
```
$npm install hexo-deployer-git --save
$hexo clean
$hexo d -g #发布
```

4.访问 https://andwong.github.io/ #查看

「疑问 : 换台电脑后如何发布博客?」
上述步骤执行完后master主支就有相应的内容,
此时新建一个分支blog用于存放博客内容,
新设备只需pull blog分支修改博客并push就行.
如果$hexo d -g 失败可以尝试删除根目录下的.deploy_git目录再执行$hexo d -g

————–分割线————
在_config.yml中修改:
title: Wong Blog #修改页面标题
author: Wong #修改作者
theme: hexo-theme-aiki #修改主题样式
