---
title: 一句话实现AVPlayer视频播放
date: 2017-05-01 18:28:57
categories: "iOS"
tags:
- Objective-C
description: 由于近期项目和视频相关的比较多，而项目中别人封装的不甚满意，所以自己进行了封装，希望有更好的扩展性，同时希望大家多多提取意见，以便于更好的封装。
---

**源码地址：**Github    https://github.com/enamor/NIPlayer

*基于AVPlayer封装的视频播放器 、一句话即可实现视频的播放 支持横屏、竖屏，监听屏幕旋转，上下滑动调节音量、屏幕亮度，左右滑动调节播放进度，快进画面预览等*

**使用说明：**

*播放器需要传入一view 自动适应view的尺寸 为了简化全屏模式统一使用屏幕旋转的方式进行适配全屏、目前控制层UI未做详细拆分，后期将逐步优化、只为做最简单的视频播放器*

~~~objective-c
//此次一句话即可实现播放 同时适配横竖屏、竖立的视频
[[NIPlayer sharedPlayer] playWithUrl:_url onView:playView];
~~~



~~~objective-c
//对播放器内部对以下状态做了监听，可以更好的自己处理各种情况
typedef NS_ENUM(NSInteger, NIAVPlayerStatus) {
    NIAVPlayerStatusLoading = 0,     // 加载视频
    NIAVPlayerStatusReadyToPlay,     // 准备好播放
    NIAVPlayerStatusIsPlaying,       // 正在播放
    NIAVPlayerStatusIsPaused,        // 已经暂停
    NIAVPlayerStatusPlayEnd,         // 播放结束
    NIAVPlayerStatusCacheData,       // 缓冲视频
    NIAVPlayerStatusCacheEnd,        // 缓冲结束
    NIAVPlayerStatusPlayStop,        // 播放中断 （多是没网）
    NIAVPlayerStatusItemFailed,      // 视频资源问题
    NIAVPlayerStatusEnterBack,       // 进入后台
    NIAVPlayerStatusBecomeActive,    // 从后台返回
};
~~~



**预览：**

![](niplayer/showhow1.png)



![](niplayer/showhow3.png)



![](niplayer/showhow4.png)