---
author: 花森煜米
layout: post
title: "使用 github 快速构建个人博客"
date: 2017-12-10
comments: true
categories: web 综合
tags: [github]
---

通常，在我们做一个项目，难免会踩不少坑，把它们记录下来好处多多。可以方便日后遇到同一个问题时，能够快速回顾之前的解决思路，不用再从头摸索，毕竟时间就是金钱。除了使用博客园、CSDN 等外，我们还想构建自己的博客，它让我们有一个属于自己的家园。好啦，blablabla，下面开始实现：

## 一、使用 Hexo 生成博客骨架

### 配置好 github 环境

首先在 github 上创建一个库来放置你的博客文件，在这里花森煜米创建了 blog 仓库，然后通过本地 git 获取远程仓库，创建 gh-pages 分支，并在远程仓库的设置中，选择 `gh-pages branch` 作为 `Github Pages` 的构建文件。

    // 克隆远程仓库到本地，这里默认的是主分支 master branch，存储博客源文件
    git clone [https://github.com/Jecyu/blog.git]
    // 新建分支 gh-pages，用于存储博客框架生成网站文件 　
    git checkout gh-pages
    // 推送到远程仓库中
    git push -u origin
 

### 二、使用 hexo 生成博客骨架

在这里假设你已经在本地电脑上配置好 [node.js](https://nodejs.org/en/) 环境，因为 Hexo 是基于 node 的静态博客生成框架。

    # 全局安装 hexo 构建文件，这里也可以选择 cnpm 国内淘宝源。
    $ npm install -g hexo-cli 
    # 执行初始化命令
    $ hexo init blog
    # 进入 blog 文件夹
    $ cd blog
    # 安装相关的依赖文件
    $  npm install

初始化成功后，blog 的目录文件结构如下

	.
	├── _config.yml  // 这里是网站的配置文件
	├── package.json
	├── scaffolds
	├── source 
	|   ├── _drafts // 草稿
	|   └── _posts // 发布的文章
	└── themes // 网站的主题

现在打开 `_config.yml` 文件，配置好远程 github 地址和部署的设置
	# URL
	url: https://jecyu.github.io/blog
	root: /blog/

    # Deployment
	deploy:
	  type: git
	  repo: git@github.com:Jecyu/blog.git
	  branch: gh-pages


通常我们在写完一篇博客后，先通过 `hexo server` 在本地预览博客网站，如果你对当前的主题不满意，你可以更改 `_config.yml` 里的主题文件，这里花森煜米使用的是 [hexo-theme-next](https://github.com/iissnan/hexo-theme-next)主题。

## 三、选择第三方服务存储静态图片、视频

当你想要说明一个很抽象的概念时，图片一目了然。当你的博客网站图片很多的时候，我们就需要第三方云服务如[`七牛云`](https://www.qiniu.com/)，个人实名认证后，官方提供 10G 的免费流量，通常对于普通的个人博客网站已经足够了，后期若需要提升再购买付费服务即可。一切准备好了，使用 `Markdown` 或其他语言开始你的写作吧，然后把它存储到 posts 目录中，使用 `hexo generate` 生成静态文件后，通过 `hexo deploy` (这里的本地分支为 master 分支)后，一个在线的个人博客网站生成。  

## 四、进一步阅读

- [hexo 官方文档](https://hexo.io/docs/index.html)

(完)