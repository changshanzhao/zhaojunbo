---
title: git常用指令
date: 2023-05-02 18:29:17
categories:
	- 教程
---

# 最常用：

```shell
// 克隆远程代码下来本地
-git clone xxxx
// 修改的代码细节展示
-git diff 
// 当前分支状态（改动总览）
-git status
// 会监控工作区的状态树，使用它会把工作时的所有变化提交到暂存区，
// 包括文件内容修改(modified)以及新文件(new)，但不包括被删除的文件。(这个说法不正确，实验证明删除文件也是可以提交的)
// git版本在2.x之后应该是把 . 和 -A作用相同化了 而不是晚上复制的
-git add .
-git add -A
// 提交暂存区文件到本地仓库
-git commit -m "我的修改备注"
// 提交本地数据到 对应的远程分支
-git push
// 查看本地对应远程的分支对应关系
-git branch -vv 
// 查看本地和远程所有分支
-git branch -a 
// 以当前本地分支作为基础新建一个xxx分支（默认你这个xxx分支 也是push到 当前分支的远程分支）
-git checkout -b xxx 
// 提交本地分支代码到 xxx远程分支
-git push origin xxx 
// 将本地分支与远程xxx分支进行关联形成关联关系
-git branch --set-upstream-to=origin/xxx 
// 拉取最新远程代码
-git pull
// 合并分支
-git merge xxxx
#这个是已经有的分支进行checkout
-git checkout xxxx
#修改的部分代码清理掉不修改了
-git checkout .
部分常用
-git branch -d xxx 删除分支（当前分支不能为xxx）
-git push origin --delete xxx(删除远程分支)
// 文件退出暂存区，但是修改保留：
-git reset --mixed
// 撤销所有的已经 add 的文件：
-git reset HEAD .
// 撤销某个文件或文件夹：
-git reset HEAD filename
// 撤销commit 之后返回成暂存区add状态
-git reset --soft HEAD^
// 撤销commit 直接新增代码全部撤销并没有add暂存直接消失
-git reset --hard HEAD^
解释：
HEAD^ 表示上一个版本，即上一次的commit，几个^代表几次提交，如果回滚两次就是HEAD^^。
--soft
不删除工作空间的改动代码 ，撤销commit，不撤销add
--hard
删除工作空间的改动代码，撤销commit且撤销add
git stash save "save tag" (贮藏已经修改的代码，如果写错分支了，没有提交 可以使用，之后切换到你需要的分支进行提取出来)
git stash list 查看贮藏的修改
git stash pop 释放贮藏内容到当前分支
// 多个贮藏 可以选择你需要拉取哪个贮藏
git stash pop stash@{$num}
git config --list 查看这个项目的git 配置
git remote prune origin xxx 这个是修剪掉已经删除远程分支的本地分支
git log 查看最近的提交信息
git reset --hard xxxxx回退到某一个历史节点
如果改动回到较远的一个节点 git push 可能会失败报错，因此我们需要强推到一个版本的话 需要：
-git push -f -u // -u 这里是为了持续推送到指定分支 这里意义不大
//下面两个命令参照这个网址 https://www.cnblogs.com/ellen-mylife/p/12794245.html
-git rebase xxx
-git rebase --continue
```



# 思考小练习（学长布置）：

1、按github文档提示，生成ssh密钥对并为自己的github账号添加公钥

2、不pull仓库，不复制文件、不在github网页上传文件，用更改remote的方式把已有文件夹里的项目上传

3、手动在网页端和本地制造冲突，并在地址pull后，用记事本和git命令行将冲突合并

4、用git rebase -i合并（fixup或者squash）几个commit

5、在3基础上，试用git rebase解决冲突，看看它对commit hash（在github上看到的7位标志或者git log中看到的）的影响以及它对代码的影响

6、在4和5操作rebase前后，查看git reflog，结合《【【中文字幕】Linus 在 2007 年 Google Talk 上介绍 Git-哔哩哔哩】 https://b23.tv/UWxTL5u 》理解git的存储方式

7、尝试用--amend方式修补上一个commit。并结合对4的理解，思考如何修补以前的commit，以及这两种修补带来的影响

8、用git revert逆转一个提交。说说这样做的意义

# 推荐网页：

https://www.yiibai.com/git
