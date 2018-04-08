---
title: iOS音视频学习
date: 2016-03-15 20:50:41
tags:
---

**音视频学习资料整理-----编码篇（VideoToolBox）**

由于项目是直播项目所以闲暇之余需要学习一下音视频编解码相关的知识。

目前iOS音视频编解码主要有两种编解码框架FFmpeg和苹果原生框架VideoToolBox 自iOS8以后开放，FFmpeg是基于CPU进行编解码的，俗称软编码，而VideoToolBox是基于GPU进行编解码的，俗称硬编码。这里我主要研究的是苹果开放的VideoToolBox。
以下是对我学习视频编码学习博客做一个整理记录，技巧只有一个就是多看多写，不懂就查：

1. [雷神音视频基础入门](http://blog.csdn.net/leixiaohua1020/article/details/18893769)
2. [音频与视频基本原理](http://blog.csdn.net/leixiaohua1020/article/details/28114081)
3. [H264(NAL简介与I帧判断)](http://www.cnblogs.com/yjg2014/p/6144977.html)
4. [移动直播技术秒开优化经验](http://www.cnblogs.com/yjg2014/p/6127454.html)
5. [如何快速的开发一个完整的iOS直播app(原理篇)](http://www.cnblogs.com/Amoyios/p/5832953.html)

全都看一遍这样对视频编解码有了一个大概的了解

还有就是对一些关键字的了解[ 码流 / 码率 / 比特率 / 帧速率 / 分辨率 / 高清的区别 ](http://blog.csdn.net/xiangjai/article/details/44238005)

掌握了一定的基础知识后 就是理论结合实际啦，代码链接地址[音视频编解码代码](https://github.com/loyinglin/LearnVideoToolBox)

以上都是纯理论只是现在进入真正的代码解析根据博客

1. [使用VideoToolbox硬编码H.264](http://www.jianshu.com/p/37784e363b8a)
2. [VideoToolBox基本函数使用](http://www.tuicool.com/articles/22A7na3)