---
title: git
tags:
---

git commit 回退

git reset --soft|--mixed|--hard <commit_id>

--mixed 会保留源码,只是将 git commit 和 index 信息回退到了某个版本.
--soft 保留源码,只回退到 commit 信息到某个版本.不涉及 index 的回退,如果还需要提交,直接 commit 即可.
--hard 源码也会回退到某个版本,commit 和 index 都会回退到某个版本.(注意,这种方式是改变本地代码仓库源码)

更换远程地址

git remote set-url origin https://gitee.com/yunbin_product/jellyfish.git

切换本地默认分支

git push --set-upstream origin browser
