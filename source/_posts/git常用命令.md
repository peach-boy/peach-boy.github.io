---
title: git常用命令
date: 2019-06-18 16:57:14
categories: git
tags: 工具
---

## git小记

###  创建分支

 1. git checkout -b develop_XXX origin/develop （以远程的develop分支为基准建立自己的本地分支）
 2. git push origin develop_XXX（将本地develop_XXX分支提交到远程）
 
### 删除分支
 
 1.git branch -D branchName（删除本地分支）
 2. git push origin --delete branchName （删除远程分支）

<!--more-->

### merge单个commit（branch_A to branch_B）
先checkout到branch_A , 从branch_B merge单个commit到branch_A 
git checkout branch_A 
git cherry-pick commitId

### git-更改本地和远程分支的名称
git branch -m old_branch new_branch # Rename branch locally 
git push origin :old_branch # Delete the old branch 
git push --set-upstream origin new_branch # Push the new branch, set local branch to track the new remote

### 回退到指定版本
git reset --hard commitId
强制提交到master分支
git push -f -u origin master


### 本地项目推送到远程分支
git init：初始化本地仓库
git add ：添加到暂存区
git commit：将暂存区修改提交到本地仓库：
git remote add git@github.com.xxxxxx:与远程仓库建立连接
git checkout develop -b(创建本地分支)
git push origin develop:推送到指定远程分支(没有分支线)


 