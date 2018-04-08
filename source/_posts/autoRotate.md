---
title: iOS状态栏旋转问题
date: 2016-06-08 01:00:30
categories: "iOS"
tags:
- Objective-C
description: iOS状态栏无法旋转解决方案
---

**1、window.rootViewController 为VC的时候 下面两种情况改方法仍须在VC中重写**

~~~objective-c
//控制器中重写
- (BOOL)shouldAutorotate {
    return NO;
}
~~~

**2、window.rootViewController 为nav的时候**

~~~objective-c
//UINavigationController 子类或者分类中重写一下方法
- (BOOL)shouldAutorotate {
    return [self.visibleViewController shouldAutorotate];
}

- (UIInterfaceOrientationMask)supportedInterfaceOrientations {
    return [self.visibleViewController supportedInterfaceOrientations];
}


- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation {
    return [self.visibleViewController preferredInterfaceOrientationForPresentation];
}
~~~

**3、window.rootViewController 为tabBar的时候**

~~~objective-c
//UITabBarController 子类或者分类中重写一下方法
- (BOOL)shouldAutorotate {
    return self.selectedViewController.shouldAutorotate;
}

- (UIInterfaceOrientationMask)supportedInterfaceOrientations {
    return self.selectedViewController.supportedInterfaceOrientations;
}

- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation {
    return [self.selectedViewController preferredInterfaceOrientationForPresentation];
}

- (UIStatusBarStyle)preferredStatusBarStyle {
    return [self.selectedViewController preferredStatusBarStyle];
}
~~~



**如果想更改状态栏的颜色需要一下设置**

需要在info.Plist 添加 View controller-based status bar appearance 设置成No，默认为Yes