---
title: CocoaPods安装流程
date: 2016-03-08
categories: "iOS"
tags:
- Objective-C
---



*2018-04-08 电脑格式化了重装CocoaPods、更新CocoaPods安装步骤，很多次安装pod setup很慢经以下步骤安装成功切安装速度可以*

## 如何下载和安装CocoaPods？

- #### 第一步：安装ruby

不能一上来就换ruby源，虽然mac自带了ruby，但是版本一般比较低，如果不进行更新会导致版本依赖问题。

这里使用rvm来管理ruby，它允许共存多个ruby。RVM：Ruby Version Manager，Ruby版本管理器，包括Ruby的版本管理和Gem库管理。

通过以下命令来安装rvm：

```
$ curl -L get.rvm.io | bash -s stable  
$ source ~/.bashrc  
$ source ~/.bash_profile  

```

完成后就可以通过以下命令来查看rvm是否安装成功：

```
rvm -v  

```

然后就可以用rvm来安装或升级Ruby了，首先查看一下Ruby的版本：

```
$ rvm list known
# MRI Rubies
[ruby-]1.8.6[-p420]
[ruby-]1.8.7[-head] # security released on head
[ruby-]1.9.1[-p431]
[ruby-]1.9.2[-p330]
[ruby-]1.9.3[-p551]
[ruby-]2.0.0[-p648]
[ruby-]2.1[.10]
[ruby-]2.2[.7]
[ruby-]2.3[.4]
[ruby-]2.4[.1]
ruby-head

```

可以看到当前的最新版本，然后通过以下命令来安装它：

```
rvm install 2.4  

```

安装好后将它设为默认版本：

```
rvm use 2.4 --default  

```

- #### 第二步：更改ruby源

升级好最新的ruby之后就可以更改ruby源了。

首先移除原有的墙外的源：

```
gem sources --remove https://rubygems.org/  

```

然后添加目前最新的由ruby官方中国维护的源：

```
gem sources -a https://gems.ruby-china.org/ 

```

然后确保当前只有这么一个源：

```
$ gem sources -l
*** CURRENT SOURCES ***  

https://gems.ruby-china.org/ 

```

然后进行ruby的组件更新：

```
sudo gem update --system  

```

以上是多数网上比较权威的步骤，如果你跟着弄下来没有问题就没有问题了。

我跟着做下来的时候在update里一直会报这样一个错误：

```
ERROR:  While executing gem ... (OpenSSL::SSL::SSLError)  
hostname "upyun.gems.ruby-china.org" does not match the server certificate 

```

最后各种尝试。用以下方法成功继续下去，如果你也有同样的情况可以尝试以下。
先把刚添加的源去掉，在重新添加一个源，把原地址最后的斜杠去掉：

```
gem sources --remove https://gems.ruby-china.org/  
gem sources -a https://gems.ruby-china.org 

```

- #### 第三步：安装CocoaPods

```
sudo gem install -n /usr/local/bin cocoapods  
sudo xcode-select --switch /Applications/Xcode.app 

```

在终端输入如下命令来完成安装：

```
pod setup  

```

这一步需要些时间，耐心等就是了。
如果安装失败，~/.cocoapods里面是空的，就需要重新setup

```
pod repo remove master  
pod setup  

```

最后如果出现Setup completed则说明安装完成了。

- #### 第四步：创建Podfile

1、新建工程，并cd到工程目录
2、新建Podfile文件

```
vim Podfile

```

3、按 i （英文输入状态下）进入编辑状态
4、输入相应的第三方和版本，比如：

```
platform:ios,'8.0'  
target '你的项目名称' do  
pod '类库名称'  
end  

```

为了确定第三方开源类库是否支持CocoaPods，可以用CocoaPods的搜索功能验证一下。在终端中输入：

```
$ pod search 类库名称

```

5、编辑好，先按esc键，再输入:wq（英文输入状态下）保存退出

6、导入第三方库

```
pod install

```

7、打开后缀为.xcworkspace的工程文件，以后编码也是在此文件中进行操作。

8、在需要使用第三方库的时候，导入头文件即可，比如：#import <AFNetworking.h>
