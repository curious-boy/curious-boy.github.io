---
layout: post
title: 有用的git命令
date: 2015-10-10 16:53:30
category: "git"
---

# git如何强制更新并覆盖本地文件 #
    git fetch --all
    git reset --hard origin/master

# 使用svn checkout github上的代码，update_editor.c出现致命错误 #
    解决方法：安装Slik-Subversion，并将程序路径加入到环境变量
