---
title: iOS组件化一【建立cocoaPods库】
date: 2018-11-24 11:59:58
tags: iOS
---



**iOS组件化学习之路一 【发布cocoaPods】**

由于目前已经做过了好多款项目，积累了一些公用的工具类、分类、UI组件等、考虑到以后项目快速导入、以及组件化的实践、特此进行cocoaPods的探究

自己的公共组件可以放到gitlab 或者 码云上来进行私有库的管理

本文讲的是自己写的一个txt阅读器发布cocoaPods的过程：

1、在github上创建一个自己的公有仓库，并上传源码

2、在终端，cd到你的项目文件目录中，创建podspec文件：

```
pod spec create CCReader-swift
```

文件名最好与你的库名字一致，库名字最好不要与pods上其他库名字重复

创建成功后，项目文件目录下会多一个xxxxx.podspec的文件，这个相当于pods仓库的配置文件，配置内容具体如下：

~~~
$ edit CCReader-swift.podspec
~~~

```
# Pod::Spec.new do |s|
s.name             = "CCReader-swift"    #名称
s.version          = "0.0.1"             #版本号
s.summary          = ".txt reader"     	#简短介绍
s.description  = <<-DESC
  .txt reader for  ios				   #详细描述
                   DESC

s.homepage     = "https://github.com/enamor/CCReader-swift.git"                          
#主页,这里要填写可以git访问到的地址，不然验证不通过
# s.screenshots     = "www.example.com/screenshots_1", "www.example.com/screenshots_2"           
#截图

s.license          = 'MIT'              #开源协议

s.source       = { :git => "https://github.com/enamor/CCReader-swift.git", :tag => "#{s.version}" }   
#项目地址，这里不支持ssh的地址，验证不通过，只支持HTTP和HTTPS，最好使用HTTPS

s.platform     = :ios, '8.0'            #支持的平台及版本


# s.author           = { "wtlucky" => "wtlucky@foxmail.com" }                   #作者信息
# s.social_media_url = 'https://twitter.com/<twitter_username>'                       #多媒体介绍地址


# s.requires_arc = true                   #是否使用ARC，如果指定具体文件，则具体的问题使用ARC

s.source_files  = "CCReader-swift", "CCReader-swift/Reader/**/*.{h,m,swift}"
#代码源文件地址， //  CCReader-swift 为项目名称    项目名称/源码路径/**/*.{h,m,swift}
```

```
# s.resource_bundles = {
# 'CCReader-swift' => ['Pod/Assets/*.png']
# }                                       #资源文件地址
# s.public_header_files = 'Pod/Classes/**/*.h'   #公开头文件地址
# s.frameworks = 'UIKit'                  #所需的framework，多个用逗号隔开
# s.dependency 'AFNetworking', '~> 2.3'   #依赖关系，该项目所依赖的其他库，如果有多个需要填写多个s.dependency
# end
```

然后将代码提交。

这个配置文件的配置内容很重要，配置的对错，会决定你的代码能否成功的推送到pods

3、为你的code打上tag版本号

```
git tag '0.0.1'
git push --tags
```

版本号自己确定，与podspec配置文件内的版本号 s.version 相匹配

4、spec配置文件检查

```
pod spec lint Peanut.podspec
```

如果不执行第三部会出现以下错误

- ERROR | [iOS] unknown: Encountered an unknown error ([!] /usr/bin/git clone https://github.com/enamor/CCReader-swift.git /var/folders/wr/gy6rn23x3vjgq81nyp493xzw0000gn/T/d20181125-2550-1yinelx --template= --single-branch --depth 1 --branch 0.0.1

5、如果是swift语言开发的会出现如下提示

the validator used Swift 3.2 by default because no Swift version was specified.

同时可能会出现如下等语法错误

- ERROR | [iOS] xcodebuild:  /Users/zhouen/Desk。。。。

那么究竟可以如何修复这个问题呢？CocoaPods 为此发布了一个修正版。在 lint 时你被要求必须指定 Swift 版本。为了这么做，你必须得创建一个名为 `.swift-version` 的新文件，并添加编译器版本。只需简单地输出以下命令： 

*在项目目录下进行如下命令*

```
echo "4.2" >> .swift-version
```

现在再次运行 `pod lib lint`，你应该能够得到一条通过验证的消息：

4、注册trunk

```shell
pod trunk register xxxx@xxxx.com 'xxxx' --description='xxxx' --verbose

/**
xxxx@xxxx.com一个可用的邮箱，注册成功之后会收到一份邮件需要点开
--description='xxxx'描述
--verbose如果出现错误，显示更多信息
*/

```

```
查看自己的信息：
pod trunk me
```

添加其他维护者

```
pod trunk add-owner xxxxxxx xxxxx@xxxx.com

/**
	xxxxxxx 库的名字
	xxxxx@xxxx.com 其他人的trunk邮箱
*/
```

 

5、提交到CocoaPods trunk

```
pod trunk push --allow-warnings
```

该操作会对本地的代码及podspec配置文件进行检查，允许警告WARN但不允许报错ERROR

也可以自己手动检查

```
pod lib lint --allow-warnings
```

 如果是在项目文件目录下pod lib lint后面不需要跟xxxxx.podspec文件名,--allow-warnings是允许警告

--------------------------------------------------------------------------------

 🎉  Congrats

 🚀  CCReader-swift (0.0.1) successfully published

 📅  November 25th, 21:55

 🌎  https://cocoapods.org/pods/CCReader-swift

 👍  Tell your friends!

\--------------------------------------------------------------------------------

出现上面情况证明已经成功



6、上面的步骤成功之后

```
pod search 你的库名
```

就能搜索到了

6、接下来是更新代码，然后推送到pods的流程。

我的代码库里只是一些零散的文件，由于推送到pods的代码不能报错，所以我这里是新建了一个空项目，然后把公共依赖的文件放到一个文件夹内Classes(名字随意)，拖到项目中，在提交的时候先编译一下，确保没有报错。

这样做的同时podspec里

```
s.source_files的配置也要改为指定的文件目录：项目名/Classes/**/*.{h,m,swift}
```

本地文件改动后，提交到git

7、然后重复步骤3，为你的代码打上tag，并推送tag（tag不能重复）

8、然后步骤5推送到trunk，这一步检查的时候很容易失败，报出的错误可以翻译过来针对性的找一下问题，基本意思都很明确

以后在每个项目中之需要在Podfile文件中增加自己的库，然后

pod install

每次有更新 pod update 你的库名