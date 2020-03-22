---
title: 博客项目构建
date: 2019-05-27 21:42:18
tags: 博客项目构建
---


## 忙活了一下午，终于使用 hexo cli 成功构建了第一个blog。记录下过程：

### 第一步：环境依赖
		node.js
		npm包管理工具
		hexo

主要是hexo的安装：

``` bash
$ npm install -g hexo-cli
```
<!-- more -->

### 第二步：项目初始化
``` bash
$ hexo init
```

## 到到这一步，已经完成了环境的基本配置。其他使用只需要了解这几个命令就可以了。

	hexo new/n "page name" //新建文章
	hexo generate/g  //页面生成
 	hexo server/s  //本地服务启动预览，localhost:4000
 	hexo deploy/d  //上传到服务器（git等等）

deploy配置的配置信息在_config.yml 中deploy

如果deploy git报错，安装deployer即可
 ```
 $ npm install hexo-deployer-git--save
 ```