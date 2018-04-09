---
title: Git命令行操作步骤
date: 2018-01-09 10:51:04
tags:
- Git
---

**Git base operate**

~~~shell
1. 创建目录文件夹并进入 pwd可查看现在所在的位置
mkdir folder

2. git创建
git init

3. 说明文档创建
touch README.md

4. 将改动添加到暂存区
git add.

5. 添加提交说明
git commit -m ‘版本1’

6. 将本地更改推送到远程分支
git push -u origin master

~~~



****

**Git branch**

~~~shell
git branch -r 查看远端分支

git branch -a 查看本地及远端分支

git checkout -b master 切换并创建分支

git branch 查看本地分支

~~~



****

**Git global setup**

~~~shell
git config --global user.name "zhouen"

git config --global user.email "xxxx@163.com"

~~~



****

**Create a new repository**

~~~shell
git clone http://192.168.1.134/zhouen/test.git

cd test

touch README.md

git add README.md

git commit -m "add README"

git push -u origin master

git pull origin master

git clone -b master http://192.168.1.134/zhouen/test.git  克隆某个分支到本地

~~~



****

**Existing folder**

~~~shell
cd existing_folder

git init

git remote add origin http://192.168.1.134/zhouen/test.git

git commit -m "Initial commit"

git push -u origin master

git add .   添加变化的内容包括新文件但不包括被删除的文件 到暂存区域

git add —all   添加变化的内容包括新文件同时包括删除的文件 到暂存区域

git remote -v

git remote remove remote_name

git remote set-url remote-name new_url

~~~



****

**Existing Git repository**

~~~shell
cd existing_repo

git remote add origin http://192.168.1.134/zhouen/test.git

git push -u origin --all

git push -u origin --tags

~~~



