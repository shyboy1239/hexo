---
title: windows下github pages + hexo next 搭建个人博客
date: 2016-08-10 11:37:24
tags: hexo
---
### 一、github pages
搭建个人博客一般需要购买域名和空间，github pages为我们提供了这两样东西，而且是免费的，相关介绍和使用方法参考这里 [github pages](https://pages.github.com/)。<!-- more -->


### 二、Hexo
一个静态博客生成框架工具，基于node.js开发。
1. 安装nodejs，[下载地址](https://nodejs.org/en/download/)。
2. 安装hexo
``` bash
$ npm install -g hexo-cli
```
3. 新建
``` bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```
4. 启动
``` bash
$ hexo server
```
 查看效果http://localhost:4000/


### 三、Next主题
一款简洁易用的hexo主题。
1. 下载安装主题
``` bash
$ cd <folder>
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```
2. 启用主题
打开站点配置文件（folder目录下_config.yml），找到theme字段，并将其值更改为next，重启之后查看效果。


### 四、配置ssh key 连接 github
1. 安装git
[下载地址](https://git-scm.com/download)
2. 生成密钥
在git-bash中输入：
``` bash
$ ssh-keygen  -t rsa –C "youremail@example.com"
```
 以rsa方式加密生成公钥id_rsa.pub私钥id_rsa
3. 添加密钥至github
将之前生成的is_rsa.pub中的内容添加至：
github -> shyboy1239. github.io -> settings -> deploy keys -> add deploy key
4. 连接测试
``` bash
$ ssh -T git@github.com
```
5. 存在多个ssh时需添加、配置config文件（可选）
``` bash
Host github.com
  HostName github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa
Host githubpages.com
  HostName github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa2
```


### 五、部署至github
1. 安装hexo-deployer-git
``` bash
$ npm install hexo-deployer-git --save
```
2. 配置deploy
打开站点配置文件（folder目录下_config.yml）,找到deploy字段，设置如下：
``` bash
deploy:
  type: git
  repo: git@githubpages.com:shyboy1239/shyboy1239.github.io.git
  branch: master
```
 *备注：因为本地存在多个ssh key,githubpages.com是之前为了进行区分而配置的别名*
3. 生成
``` bash
$ hexo generate
```
4. 部署
``` bash
$ hexo deploy
```


### 六、结语
最后，一个基本的个人博客就搭建好了！~
