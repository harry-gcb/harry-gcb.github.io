---
layout: title
title: git submodule子模块
date: 2023-04-24 21:59:43
tags: [git]
---

项目中经常使用第三方模块，使用git子模块可以将第三方git仓库作为一个子目录，在保持独立提交的同时，不必负责子模块的维护，只需要在使用的时候同步更新子模块即可，能够大大提升开发效率。

本文是对git子模块命令的一个简介，详细使用参考man page。

## 添加子模块

添加子模块的命令非常简单：`git submodule add <url> <path>`，其中`url`为远程仓库地址，`path`为子模块的本地目录。如下面的例子：

```shell
git submodule add git@github.com:harry-gcb/hexo-theme-butterfly.git themes/butterfly
```

执行成功后，可以看到`themes`下面多了`butterfly`目录，这就是刚下载的子模块。

添加子模块后，如果运行`git status`，可以看到新增了`themes/butterfly`和`.gitmodules`，`.gitmodules`则用来保持子模块的信息。`.gitmodules`中的内容如下

```shell
[submodule "themes/butterfly"]
	path = themes/butterfly
	url = git@github.com:harry-gcb/hexo-theme-butterfly.git
```

其中`path`和`url`即使添加子模块使用的参数。

`git diff --cached`查看修改内容可以看到增加了子模块，并且新文件下为子模块的提交hash摘要

`git commit`提交即完成子模块的添加

## 更新子模块

更新项目内子模块到最新版本`git submodule update`

更新子模块为远程项目的最新版本`git submodule update --remote`

或者进入子模块目录进行`git pull/push`等操作

## 删除子模块

删除子模块比较复杂，因为子模块的维护地址可能发生变化，或者需要替换子模块，这时就需要删除原有的子模块

1. 删除子模块缓存

   ```
   git rm --cached themes/butterfly
   ```

2. 删除子模块文件夹

   ```
   rm -rf themes/butterfly
   ```

3. 删除`.gitmodules`中关于子模块的信息

   ```
   vim .gitmodules
   #[submodule "themes/butterfly"]
   #	path = themes/butterfly
   #	url = git@github.com:harry-gcb/hexo-theme-butterfly.git
   ```

4. 删除`.git/config`中关于子模块的信息

   ```
   vim .git/config
   #[submodule "themes/butterfly"]
   #	url = git@github.com:harry-gcb/hexo-theme-butterfly.git
   #	active = true
   ```

5. 删除`.git/modules`中子模块对应的目录

   ```
   rm -rf .git/modules/themes/butterfly
   ```

## clone包含子模块的项目

如果只是用`git clone`克隆一个包含子模块的项目时，默认子模块目录下可能没有任何内容，需要执行一下命令下载子模块

```shell
git submodule init
git submodule update
```

或者

```
git submodule update --init --recursive
```

如果是第一次`git clone`下载主仓库的项目，也可以使用以下命令将主仓库和子模块的所有内容都下载下来

```shell
git clone --recursive <url>
```

以后可以在子模块仓库中使用`git pull`和`git push`等命令进行更新合并等操作

## 参考

* [git中submodule子模块的添加、使用和删除](https://rouroux.github.io/git-submodule/)

* [Git: submodule 子模块简明教程](https://iphysresearch.github.io/blog/post/programing/git/git_submodule/)
* [Git 工具 - 子模块](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)
