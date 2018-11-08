---
title: git 远程存在文件本地不想提交
date: 2018-01-05 10:16:33
categories: GIT
tags: GIT
---
开发中会遇到一些本地和远程同时共用文件,本文件不想提交,但是添加.gitignore 忽略时会删除远程文件

此时我们需要使用以下命令来添加本地忽略文件

以下命令作用可以保证远程和本地存在同一个文件但是不会互相影响



添加本地忽略文件

git update-index --assume-unchanged FILENAME

 

恢复本地忽略文件

开发中会遇到一些本地和远程同时共用文件,本文件不想提交,但是添加.gitignore 忽略时会删除远程文件

此时我们需要使用以下命令来添加本地忽略文件

以下命令作用可以保证远程和本地存在同一个文件但是不会互相影响



添加本地忽略文件

git update-index --assume-unchanged FILENAME

 

恢复本地忽略文件

git update-index --no-assume-unchanged FILENAME