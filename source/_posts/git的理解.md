---
title: git的理解
date: 2019-02-15 00:01:32
tags: git
---

# Git 的配置

Git 的配置分三级：系统级（system），全局配置（global），版本库级（local）

- 系统级（system）：包括所有用户及其仓库的值。对应配置在文件 */etc/gitconfig* 中

- 全局配置（global）：针对的是用户的配置。对应配置在文件 *~/.gitconfig* 中。命令行配置：git config --glocal [-l][< name > < value >][< key >]

- 版本库级（local）：当前仓库的 git 目录。对应配置在文件 *.git/config* 中

配置文件的优先级：版本库级 > 全局配置 > 系统级

# 获取 Git 仓库

# 部分常用 Git 命令

- git diff：查看已跟踪文件的具体更改。默认是未暂存的文件的更改，加参数[--staged | --cached]可以查看已暂存文件的更改。 图形化界面查看可以通过命令 *git difftool* 查看，命令 *git difftool --tool-help* 可以查看系统中支持的 diff 工具。