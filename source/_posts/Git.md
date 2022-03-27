---
layout: posts
date: 2017-04-05 09:49:27
title: Git
categories: 学习笔记
tags: [Git, 版本控制器]
---

# Git

### 一、环境配置

git config --global user.name 'yourname'

git config --global user.email 'youremail'

git config --global core.autocrlf false     #由于开发环境不同，配置不要管换行转换符

<!-- more -->

git config --global core.quotepath off    #配置避免中文乱码



生成密钥

ssh-keygen -t rsa -C 'your-email'



至此：window下的环境已配置好

在linux下还要配置：

ssh-add ~/.ssh/id_rsa

当出现问题：![img](file:///C:/Users/Lifu/AppData/Local/YNote/data/qqC1328C0E098C90A13819B468B3DF0404/ba01a2b13f524be08aa84ec376802429/clipboard.png)

执行：eval 'ssh-agent'   ==注意：这里的这个'是 Esc 下面的那个==

再重新执行：ssh-add ~/.ssh/id_rsa



最后一步就是将公钥添加到你用的码云上



### 二、添加远程仓库

git remote add origin 'ssh-key'



git add .

git commit -m ''

git push -u origin  remote-branch -f  强制推送到远程分支（不建议使用，在第一次的时候可以这样）