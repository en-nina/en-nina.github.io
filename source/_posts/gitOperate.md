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
git add .

5. 添加提交说明
git commit -m ‘版本1’

6. 将本地更改推送到远程分支
git push -u origin master

7. 在git中重命名远程分支，其实就是先删除远程分支，然后重命名本地分支，再重新提交一个远程分支
git push --delete origin devel //删除远程分支
git branch -m oldName newName  //重新命名本地分支
git push -u origin master

~~~



****

**Git branch**

~~~shell
git branch -r 查看远端分支

git branch -a 查看本地及远端分支

git checkout -b master 切换并创建分支

git branch 查看本地分支

git branch -m oldName newName  //重新命名本地分支

~~~



------

**Git branch delete**

~~~shell
git push origin --delete dev  删除远程分支

git branch -a  查看分支

git branch -D dev 删除本地分支

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

git add -A   添加变化的内容包括新文件同时包括删除的文件 到暂存区域

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



------

**Ignore the file**

```shell
touch .gitignore

//例：iOS过滤第三方库文件 修改.gitignore 文件加入一下文本
.DS_Store
*.xcworkspace
xcuserdata
*.lock
Pods

//进入目录删除已经存在的git (有的文件中已经存在git导致无法提交)
rm -rf .git
```



------

**tags**

```shell
//删除tag
git tag -d 标签名  
例如：git tag -d v3.1.0

//git 删除远程标签
git push origin :refs/tags/标签名  
例如：git push origin :refs/tags/v3.1.0

//添加 tag
git tag 标签名
例如：git tag v2.0.0

//提交tag到 远端
git push origin 标签名
例如：git push origin v2.0.0
```



------

**Problems and Solutions**

1. git push -u origin master 遇到错误The requested URL returned error: 403 Forbidden while accessing

```shell
解决方案：vim .git/config
修改URL： url = https://github.com/enamor/PageControl.git
修改后：  url = https://enamor@github.com/enamor/PageControl.git

之后 再次git push，弹出框输入密码，即可提交
```

2. failed to push some refs to 'https://enamor@github.com/enamor/CCReader-swift.git'

   这个问题是因为远程库与本地库不一致造成的，那么我们把远程库同步到本地库就可以了。 
   使用指令

~~~shell
git pull --rebase origin master

//这条指令的意思是把远程库中的更新合并到本地库中，–rebase的作用是取消掉本地库中刚刚的commit，并把他们接到更新后的版本库之中。
~~~



**让 .gitignore 文件生效**

~~~
让 .gitignore 文件生效
查看.gitignore文件名是否写错
对照网上的资料查看规则有没有写错
下面是最重要的重置缓存。
git rm -r --cached . -->清除缓存
git add . -->添加缓存
git commit -m “添加过滤文件”
备注：注意你所处的分支，如果你在当前分支修改，切换到其他分支是不生效的，如果多人开发，注意合并修改！

~~~

