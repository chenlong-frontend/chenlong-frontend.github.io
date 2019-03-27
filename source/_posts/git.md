---
title: git
author: 陈龙
date: 2019-03-26 20:11:50
tags: [git]
categories: [git]
---

最近工作中在使用 git 来管理项目，以下是对于一些自己实际遇到一些坑的总结。

## 添加本地仓库

刚开始的时候我是使用`git add .`来添加修改文件到本地仓库的，但有一个由于我的疏忽，我没有在根目下执行这条指令，导致一些修改文件没有被提交。在这件事之后我改用`git add --all`来添加所有的改动，并使用`git status`查看文件是否正确。

## 新建分支

当需要对一个项目进行大改一般会新建一个分支，使用`git checkout -b 分支名称`来新建一个分支并同时切换到那个分支上，使用`git checkout 分支名称`来切换分支，使用`git merge 分支名称`将指定分支合并到当前分支。

## 切换本地默认分支

当从主分支切换到其他分支时，每次提交拉取代码时都要加上`origin 分支名称`，可使用`git push --set-upstream origin 分支名称`将指定分支切换为本地默认分支。

## git commit 回退

有时`commit`代码之后发现把无用的东西提交了，可使用`git log`打印出最近的`commit`记录，然后通过`git reset --soft|--mixed|--hard <commit_id>`来进行`commit`回退。

- mixed 会保留源码,只是将 git commit 和 index 信息回退到了某个版本.
- soft 保留源码,只回退到 commit 信息到某个版本.不涉及 index 的回退,如果还需要提交,直接 commit 即可.
- hard 源码也会回退到某个版本,commit 和 index 都会回退到某个版本.(注意,这种方式是改变本地代码仓库源码)

## 更换远程地址

遇到过一次项目远程地址更换，可使用`git remote set-url origin https://gitee.com/yunbin_product/jellyfish.git`进行更换

## git 区分大小写

记得一次我修改了某文件夹的首字母的大小写，结果提交的时候并没有产生修改记录，导致了其他人运行项目时出错，后经排查是 git 忽略了大小写，可使用如下配置正确区分大小写。

```shell
git config core.ignorecase false
```
