---
title: Github+Hexo搭建个人博客
date: 2023-04-22 14:07:03
tags: [hexo]
---

本文内容主要记录Hexo+Github搭建个人博客的过程，以及后续维护时如何添加新的feature

## 环境准备

* `node.js`
* `git`

这两个应用直接在官网下载安装即可

## 安装hexo

安装`hexo`

```
npm install hexo-cli -g
```

选择合适的目录，创建博客文件夹<folders>

```
cd <folders>
hexo init
npm install
```

## 配置github

本地配置github ssh连接，方便自动部署，具体过程略

新建仓库，仓库名必须为**github_username.github.io**

安装`hexo-deployer-git`

```
npm install hexo-deployer-git
```

修改站点配置

```
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  repository: git@github.com:harry-gcb/harry-gcb.github.io.git
  branch: main
```

## 文章编写与发布

在根目录输入`hexo new post "article title"`即可新建一篇文章，此时可以发现`_posts`下面多了个一个`article-title.md`文件。在这个markdown文件中编写文章后，输入一下命令即可生成、浏览和发布文章

* `hexo g` 生成静态网页
* `hexo s`打开本地网页预览效果
* `hexo d`部署到github，打开`github_username.github.io`即可浏览文章

## 主题安装

hexo的主题安装及其简单，选择一款喜欢的主题下载到`themes`目录下，然后将根目录`_config.yml`中的`theme`设置为自己的主题即可。我选择使用`butterfly`这款主题，主要是文档全面，而且`github`比较活跃，有问题能及时反馈。`Butterfly`: [Butterfly官方文档](https://butterfly.js.org/)

### 

