---
title: 流式加载数据
tags: [react]
categories: [react]
---

```js
git config --global user.name "wirelessqa"
git config --global user.email wirelessqa.me@gmail.com
git config --list

git fetch origin  # 同步远程仓库

//查看本地分支
git branch

//删除目标分支
git branch -D master

//重新拉取master分支
git checkout master

//gitreset
git reset是比较危险的，使用git revert来安全的撤回

//这两者一起使用彻底恢复到某次提交
// git clean 移除项目下未被跟踪的文件
git reset --hard 和 git clean -f

// 修复提交的错误信息，修改提交历史
git commit --amend
```
