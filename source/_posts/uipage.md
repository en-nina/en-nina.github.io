---
title: UIPageViewController上下滑动页面重复Bug解决方案
date: 2018-10-12 12:07:15
categories: "iOS"
tags:
- Objective-C
---



*UIPageViewController实现小说阅读器遇到的bug及解决方案*

**上下滑动引起页面重复的bug**

~~~objective-c
解决方案：修改UIPageViewController的UIPanGestureRecognizer手势

//1 修改UIPageViewController的UIPanGestureRecognizer手势代理
    for (UIGestureRecognizer *gr in _pageViewController.gestureRecognizers) {
        if ([gr isKindOfClass:[UIPanGestureRecognizer class]]) {
            
            gr.delegate = self;
        }
    }

//实现代理方法
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch {
    
    //可如下方式过滤手势
//    if ([touch.view isDescendantOfView:self.topView] ||
//        [touch.view isDescendantOfView:self.bottomView] ||
//        [touch.view isDescendantOfView:self.settingView] ) {
//        return NO;
//    }
    
    
    if ([gestureRecognizer isKindOfClass:[UIPanGestureRecognizer class]]) {
        
        UIPanGestureRecognizer *swipe = (UIPanGestureRecognizer *)gestureRecognizer;
        if (gestureRecognizer.state == UIGestureRecognizerStateChanged) {
           return  [self commitTranslation:[swipe translationInView:self.view]];
        }
    }
    return YES;
}

//判断滑动方向
- (BOOL)commitTranslation:(CGPoint)translation {
    CGFloat absX = fabs(translation.x);
    CGFloat absY = fabs(translation.y);
    
    // 设置滑动有效距离
    if (MAX(absX, absY) < 20)
        return NO;
    
    
    if (absX > absY ) {
        
        if (translation.x < 0) {
            //向左滑动
        }else{
            //向右滑动
        }
        
        return YES;
        
    } else if (absY > absX) {
        if (translation.y<0) {
            
            //向上滑动
        }else{
            
            //向下滑动
        }
        return NO;
    }
    
    return YES;
    
}
~~~



**快速滑动 及其 滑动未完成又返回引起的问题**

~~~objective-c
#pragma mark - UIPageViewControllerDataSource
//显示前一页
- (UIViewController *)pageViewController:(UIPageViewController *)pageViewController viewControllerBeforeViewController:(UIViewController *)viewController {
    if (self.transiting) { return nil; }
    return vc;
}

//显示下一页
- (UIViewController *)pageViewController:(UIPageViewController *)pageViewController viewControllerAfterViewController:(UIViewController *)viewController {
    if (self.transiting) { return nil; }
    
    return vc;
}

- (void)pageViewController:(UIPageViewController *)pageViewController willTransitionToViewControllers:(NSArray<UIViewController *> *)pendingViewControllers {
    //防止快速滑动引起的问题
    self.transiting = YES;
}

- (void)pageViewController:(UIPageViewController *)pageViewController didFinishAnimating:(BOOL)finished previousViewControllers:(NSArray<UIViewController *> *)previousViewControllers transitionCompleted:(BOOL)completed {
    self.transiting = NO;
    
    if (!completed) {
        修改页面索引为未完成时的索引 
            
    }
}
~~~

