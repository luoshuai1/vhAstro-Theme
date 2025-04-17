---
title: Git提交本地和导出diff
date: 2022-06-29 08:54:55
categories: Code
tags:
  - git
  - bash
id: "js-gitdiff"
recommend: true
---

:::note
本文详细介绍了Git的基本操作流程，包括状态查看、代码拉取、更改暂存、代码合并、提交、导出diff、分支管理等，适合初学者快速掌握Git的使用方法。
:::

### Git提交本地和导出diff

```bash
1. git status //查看修改
2. git fetch //拉取最新代码
3. git stash //暂存更改(必须是工作区中已经被git追踪到的文件),可以使用git add 先添加跟踪在暂存
4. git rebase //合并拉取的代码，fetch到内容，暂存后要rebase。如果使用git pull 会自动合并，是git fetch 加 git merge，pull有时候合并失败一般是冲突，需解决冲突在commit 加push
5. git stash pop 取出最近一次暂存代码。 git restore --staged xxx.text 释放缓存区，保留修改，加 . 释放所有缓存区文件。git stash list 查看暂存列表。git stash apply stash@{n} 释放暂存区某条暂存
6. git add /view.index //写入某文件到缓存区。// add . 所有修改文件写入缓存区
7. git rebase --continue //解决冲突文件
8. git commit //提交到本地仓库中。 // i 插入描述内容， 按shift和:进入命令， wq 保存提交， q 退出
9. git log -2 //查看提交的最新2个日志
10. git show > g:/123.diff //导出最新本地提交的diff文件（commit）；git diff --cached  xxx  > g:/123.diff   //对比暂存区(git add 之后)和版本库(git commit 之后)
11. git show //查看最新的diff文件
12. git git commit --amend //合并到之前的一个本地提交，出现俩个本地提交时使用
13. git push //提交到git服务器仓库
14. git cherry -v //查看未push到服务器的本地提交
15. git reset 2b638b8c5195af0d1bde52afd46a09501dbe7724 //本地还原到某个提交的版本 git reset --hard 强制回退，不保留修改， git reflog 所有分支的所有操作记录 #反悔reset (30天内)
16. git reset --soft HEAD~1 //撤销上一次commit 版本，不撤销add ，改动代码。 2次commit，想都撤回，可以使用HEAD~2

```

### 分支用法

:::note
1.git branch 查看本地分支    git branch -a  查看远程分支 git remote update origin --p 更新远程分支列表  git branch -vv 查看所有本地分支对应的远程分支  

2.git branch test 创建本地分支，并切换到分支 git checkout -b test origin/远程分支名  拉取远程分支并切换到该分支

3.git checkout  xxx   切换分支

4.git merge xxx  合并分支，先切换到需合并的分支

5.git branch -d xxx  删除本地分支  git push origin --delete 分支名  删除远程分支

注意：分支的工作区和暂存区是公共的。

**clone的分支默认能push 到远程分支。新创建的分支要push，如果不写远程分支名称，则默认和本地分支同名，这时命令为:$ git push origin test    // origin 是默认的远程版本库名称**

**如果需要使用导出的patch文件恢复 code，请参考如下**
    git apply --check 0001xx.diff//检查patch是否能成功导入
    如果提示没有error，则使用 git apply 0001xx.diff
    如果提示有error，则使用强制导入git apply –reject 0001xx.diff //会冲突文件产生对应的.rej文件, 查看并参考.rej文件修改原文件，之后再删除.rej文件
:::


### 使用说明

:::md
    1. 在本地修改与远程代码无冲突的情况下，优先使用：
        **pull -> commit -> push**
    2. 在本地修改与远程代码有冲突的情况下，优先使用：
        **commit -> pull -> push**
:::


:::note
  保险起见还是先git add . git commit 再git pull比较好。

    先 commit 再 pull 再 push 的情况就是为了应对多人合并开发的情况：
    commit 是为了告诉 git 我这次提交改了哪些东西，不然你只是改了但是 git 不知道你改了，也就无从判断比较；
  pull是为了本地 commit 和远程commit 的对比记录，git 是按照文件的行数操作进行对比的，如果同时操作了某文件的同一行那么就会产生冲突，git 也会把这个冲突给标记出来，这个时候就需要先搞清楚冲突的代码保留那个版本的，然后在 git add 、 git commit 、 git pull 三连，再次 pull 一次是为了防止在这期间另一个人给又提交了一版代码，通常没有冲突的时候就直接合并了，不会把你的代码给覆盖掉 ；
:::


