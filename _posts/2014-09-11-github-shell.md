---
layout: post
title: Github常用命令
category: Github
tags: [Github]
---

将本地代码库更新为和远端一致，相当于SVN的update：

    git pull

查看本地代码库的状态，需要经常使用：

    git status

将本地代码库中添加的代码添加到待提交区，继而可以用于commit：

    git add .

在本地代码库删除代码，继而还需要commit：

    git rm xxx

将待提交区的代码提交到待传输区：
 
    git commit -m "xxx"

将待传输区的代码传输到远端代码库：

    git push origin master


-EOF-
