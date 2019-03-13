---
title: git 一些笔记
date: 2019-02-15 00:01:32
tags: git
---

# Git 的配置

Git 的配置分三级：系统级（system），全局配置（global），版本库级（local）

- 系统级（system）：包括所有用户及其仓库的值。对应配置在文件 */etc/gitconfig* 中

- 全局配置（global）：针对的是用户的配置。对应配置在文件 *~/.gitconfig* 中。命令行配置：git config --glocal [-l][< name > < value >][< key >]

- 版本库级（local）：当前仓库的 git 目录。对应配置在文件 *.git/config* 中

配置文件的优先级：版本库级 > 全局配置 > 系统级

# 部分常用 Git 命令

- git diff：查看已跟踪文件的具体更改。默认是未暂存的文件的更改，加参数[--staged | --cached]可以查看已暂存文件的更改。 图形化界面查看可以通过命令 *git difftool* 查看，命令 *git difftool --tool-help* 可以查看系统中支持的 diff 工具。

- git remote show [remote-name]：查看远程仓库URL地址以及每一个分支的跟踪信息。如果可以看到远程仓库中哪些分支已跟踪，哪些是新分支，哪些是本地有而远程已经删除了的分支，pull到本地的分支对应关系，push到远程的分支的对应关系。

- git push origin [tagname]: 共享标签，可以将标签推送到远程。如果要一次性推送所有标签的话，可以使用命令git push origin -tags。

- git branch --merge: 可以查看所有已合并到当前分支的分支。

- git branch --no-merge: 可以查看所有未合并到当前分支的分支。

# 远程分支

- 拉取：git fetch origin命令，会去查询“origin”服务器取得所有本地尚未包含的数据，然后更新到本地数据库，最后把本地的origin/master指针移动到最新位置上。拉取远程分支，如果其中有本地没有创建过的分支，并不会自动去创建这些分支，只是本地会有对应的origin/xxxx指针。

- 跟踪分支：基于远程分支创建的本地分支会自动成为跟踪分支（tracking branch），或者有时候也叫做上游分支（upstream branch）。
即：跟踪分支是与远程分支直接关联的本地分支，所以如果在一个跟踪分支上git push的话，Git可以知道该推送到远程仓库的哪个分支，git pull的时候Git知道该从哪个服务器拉取分支并合并到本地分支。
如果要给本地分支设置跟踪分支，或者更改本地分支对应的远程分支，可以使用命令 git branch -u ... 或 git branch --set-upstream-to ...

```bash

$ git branch -u origin/serverfix
Branch sercerfix set up to track remote branch serverfix from origin


```

- 查看目前本地分支的跟踪分支状态： git branch -vv

# Git工具

- git log

1. git log master..experiment: 可以查看所有“可以在experiment分支中获得，但不能在master分支获得的提交”。
(上面命令等效于：git log master --not experiment，也等效于：git log master ^experiment)
git log experiment..master: 可以查看所有“可以在master分支中获得，但不能在experiment分支获得的提交”。

2. 涉及多条分支：git log refA refB --not refC (git log refA refB ^refC) 可以查看所有“可以在refA和refB上获得，但不能从refC上获得的提交”。

- git add -p xxx
p是patch的缩写。可以通过这个命令对文件进行部分暂存，比如一个文件中有些修改想暂存，有些不像暂存，可以由此进行交互选择。

# Git调试

- git blame -L m,n filename: 可以看到文件 filename 从 m 行到 n 行的修改信息。

- git bisect: 二分查找。原来没出现过的bug，现在出现了，二分查找帮助找到问题出现的提交

```bash

git bisect start // 启动排查过程
git bisect bad // 告诉系统当前提交有问题
git bisect good [good_commit] // 告诉bisect最后一次正常状态是什么时候
// Git会在你认为最后一次正确的提交和当前错误提及哦啊之间检出中间那一次，这时可以进行测试，如果这个中间提交是有问题的，那么：
git bisect bad // 告知Git
// 如果这个中间提交是没有问题的，那么
git bisect good // 告知Git
// 排查结束后得到最开始出现问题的提交
git bisect reset // 将HEAD重置到排查之前的位置


```

什么是“fast-forward”？
当合并分支时所说的“fast-forwaord”，是指合并的两个分支，其中一个分支是另一个分支的直接上游（即父提交），此时GIT会尝试直接更改该提交的指针指向的提交，将指针直接向前移动。
如果要合并的两条分支是从某个共同祖先开叉出来的，即不是直接上游。那么Git会去判断找到它们之间最优的共同祖先，尝试基于三方（共同祖先，两个待合并分支）结果，创建新的快照。
