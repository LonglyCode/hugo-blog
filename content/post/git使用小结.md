--- 
title: git使用小结
date: 2016-01-24 23:40:05
tags: ["git", "tool"]

---
<!-- toc -->

## git 一些原理和概念

 1. 在本地有三个区域：工作区，staged 区以及本地仓库，当然还有一个抽象的概念的就是分支
 2. 远程有：远程仓库和分支
 3. HEAD 是个指针，指向 git 自动帮我们创建的第一个分支 master。master 才指向提交的，提交到远程
> 几乎所有的操作的都是基于以上的区域的，值得一提一点：本地的 git 仓库和 github 仓库之间的传输是通过 SSH 加密的

<!--more-->

## git初始化
1. git init在目录下面新建一个git 仓库

## git 添加操作
 1. git add 添加到 staged 暂存区，加入也意味着可以 track 追踪文件的变化状态，可以是**git add "file name"**或者添加整个目录**git add .**
 2. git commit 提交到本地的仓库的当前分支，加参数`-m(message)`添加提交信息，`-a(all)`则省略add操作直接提交到仓库。
 3. git push origin master 提交到远程仓库的默认库的 master 分支
 4. git push -u origin master，把本地库的内容推送到远程，用 git push 命令，实际上是把当前分支 master 推送到远程。由于远程库是空的，我们第一次推送 master 分支时，加上了-u 参数，Git 不但会把本地的 master 分支内容推送的远程新的 master 分支，还会把本地的 master 分支和远程的 master 分支关联起来，在以后的推送或者拉取时就可以简化命令。git push origin master

## git 查看操作

1. git log 查看提交历史，–pretty=oneline 每个版本信息放到一行显示，分支历史也可以用这个命令查看
2. git reflog 可以查看命令历史，re-reset 操作可以从其中找到 ID,恢复到最新版本
3. git status 查看一下现有状态，针对本地库
4. 用 git diff HEAD – file 命令可以查看工作区和版本库里面最新版本的区别，git diff什么都没加表示查看当前repository与暂存区的差别。
5. 用 git log -–graph 命令可以看到分支合并图

## git 还原操作

1. git checkout –file 其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”
2. 命令 git rm 删掉，并且 git commit，此为彻底删除操作
3. 用命令 git reset HEAD file，git reset 命令既可以回退版本，也可以把暂存区的修改回退到工作区。当我们用 HEAD 时，表示最新的版本
4. head 为当前版本，head^是上一个版本，head^^是上上个版本，用 HEAD~100 代表 100 个版本之前 git reset –hard HEAD^回到上一个版本，git reset –hard <ID>，加 ID 比较通用，HEAD 就指向那个 ID 了

## git 分支操作

1. git checkout -b dev，git checkout 命令加上-b 参数表示创建并切换 dev 分支
2. dev 分支的工作完成，我们就可以切换回 master 分支：git checkout master
3. 用 git branch 命令查看当前分支
4. git branch -d dev 用来删除 dev 分支操作
5. git merge branchname 命令用于合并指定分支到当前分支准备合并 dev 分支，请注意–no-ff 参数，表示禁用 Fast forward。通常，合并分支时，如果可能，Git 会用 Fast forward 模式，但这种模式下，删除分支后，会丢掉分支信息。比如 git merge –no-ff -m “merge with no-ff” dev
6. 如果要丢弃一个没有被合并过的分支，可以通过 git branch -D <name>强行删除
7. git-rebase +命令主要用在从上游分支获取最新 commit 信息，并有机的将当前分支和上游分支进行合并,可能会出现冲突
8. git checkout - 切换回上一个分支。

## git 远程操作

1. git clone [url] 克隆一个远程仓库到本地
2. 当你从远程仓库克隆时，实际上 Git 自动把本地的 master 分支和远程的 master 分支对应起来了，并且，远程仓库的默认名称是 origin
3. 查看远程库信息，使用 git remote -v；git remote add [url]本地仓库关联远程分支，url就是网络仓库地址如`git@github.com:LonglyCode/LonglyCode.github.io.git`；
4. 从本地推送分支，使用 git push origin branch-name，第一次推送最好加上`-u`参数；如果推送失败，先用 git pull 抓取远程的新提交；
5. 在本地创建和远程分支对应的分支，使用 git checkout -b branch-name origin/branch-name，本地和远程分支的名称最好一致；
6. 建立本地分支和远程分支的关联，使用 git branch –set-upstream branch-name origin/branch-name；
7. 从远程抓取分支，使用 git pull，如果有冲突，要先处理冲突。
8. git fetch：相当于是从远程获取最新版本到本地，不会自动 merge,= git pull 相当于 git fetch + git merge =

## git 标签操作

1. Git 的标签虽然是版本库的快照，但其实它就是指向某个 commit 的指针（跟分支很像对不对？但是分支可以移动，标签不能移动），所以，创建和删除标签都是瞬间完成的
2. 命令 git tag <name>用于新建一个标签，默认为 HEAD，也可以指定一个 commit id；
3. git tag -a <tagname> -m “blablabla…”可以指定标签信息；
4. git tag -s <tagname> -m “blablabla…”可以用 PGP 签名标签；
5. git tag 可以查看所有标签；
6. git push origin <tagname>可以推送一个本地标签；
7. git push origin –tags 可以推送全部未推送过的本地标签；
8. git tag -d <tagname>可以删除一个本地标签；
9. git push origin :refs/tags/<tagname>可以删除一个远程标签

## MISC

1. Git 还提供了一个 stash 功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作，你可以多次 stash，恢复的时候，先用 git stash list 查看，然后恢复指定的 stash，git stash pop 用来恢复，恢复的同时把 stash 内容也删了
2. 在 GitHub 上，可以任意 Fork 开源仓库；自己拥有 Fork 后的仓库的读写权限；可以推送 pull request 给官方仓库来贡献代码。
3. git config –global alias.st status，用来设置别名减少操作，现在 git st就代表了 git status了
4. ssh-keygen -t rsa -C “my@email.com” 设置SSH
