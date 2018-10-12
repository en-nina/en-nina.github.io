---
title: iOS小知识点集锦
date: 2015-06-01 15:28:57
categories: "iOS"
toc: true
tags:
- Objective-C
description: 自己平时的小知识点笔记
---

[TOC]

###iOS10 AVPlayer有时播放失败的处理

~~~objective-c
//如果你的视频使用的是HLS(m3u8)协议的，是不会由于升级ios10出现这个播放问题的。 如果不是基于HLS协议的，解决方法如下
if([[UIDevice currentDevice] systemVersion].intValue>=10){
//      增加下面这行可以解决iOS10兼容性问题了
      self.player.automaticallyWaitsToMinimizeStalling = NO;
}

/*ios10中AVPlayer增加了多个属性，其中有几个需要注意一下
	1、timeControlStatus
	2、automaticallyWaitsToMinimizeStalling
	
	正是第二个属性的默认值导致了不使用HLS的方式（比如使用localhttpserver间接实现在线播放、或者下载到本地再播放的方式）就会播放不了的问题。
*/

//在之前的版本中，我们通过rate来判断avplayer是否处于播放中
- (Boolean)isPlaying {
    return self.player.rate==1;
}

//在iOS10中，AVPlayer多了一个timeControlStatus，我们就应该这么实现
- (Boolean)isPlaying {
    if([[UIDevice currentDevice] systemVersion].intValue>=10){
        return self.player.timeControlStatus == AVPlayerTimeControlStatusPlaying;
    }else{
        return self.player.rate==1;
    }
}
~~~

### 字符串去除空格

~~~objective-c
//去除首尾空格：  
NSString * content = [textView.text stringByTrimmingCharactersInSet:
                      [NSCharacterSet whitespaceCharacterSet]];

//去除首尾空格和换行：
NSString *content = [textView.text stringByTrimmingCharactersInSet:
                     [NSCharacterSet whitespaceAndNewlineCharacterSet]];
~~~



### 控制器禁用左侧手势返回功能

~~~objective-c
//关闭手势返回
- (void)viewDidAppear:(BOOL)animated  
{  
    [super viewDidAppear:animated];  
    // 禁用返回手势  
    if ([self.navigationController respondsToSelector:@selector(interactivePopGestureRecognizer)]) {  
        self.navigationController.interactivePopGestureRecognizer.enabled = NO;  
    }  
}  
//重新开启手势返回
- (void)viewWillDisappear:(BOOL)animated  
{  
    [super viewWillDisappear:animated];  
    // 开启返回手势  
    if ([self.navigationController respondsToSelector:@selector(interactivePopGestureRecognizer)]) {  
        self.navigationController.interactivePopGestureRecognizer.enabled = YES;  
    }  
}  
~~~

### UIView 视频全屏 masonry 约束设置

~~~objective-c
    UIView *player = [[UIView alloc] init];
    [self.view addSubview:player];
    
    CGFloat wid = [UIScreen mainScreen].bounds.size.width;
    CGFloat hei = [UIScreen mainScreen].bounds.size.height;
    CGFloat rate = wid < hei ? (wid/hei) : (hei/wid);
    [player mas_makeConstraints:^(MASConstraintMaker *make) {
        make.top.equalTo(self.view);
        make.left.right.equalTo(self.view);
        make.height.mas_equalTo(player.mas_width).multipliedBy(rate);
    }];
~~~

### Masonry实现动画更新约束

~~~objective-c
// 告诉self.view约束需要更新
[self.view setNeedsUpdateConstraints];
// 调用此方法告诉self.view检测是否需要更新约束，若需要则更新，下面添加动画效果才起作用
[self.view updateConstraintsIfNeeded];

[UIView animateWithDuration:0.3 animations:^{
  [self.view layoutIfNeeded];
}];
~~~

### UISlider更改进度条的粗细 、修改圆点位置

~~~objective-c
//需要重写UISlider的trackRectForBounds 方法
- (CGRect)trackRectForBounds:(CGRect)bounds {
    bounds.origin.y = bounds.size.height / 2.0 - 2;
    bounds.size.height = 4;
    return bounds;
}

//小圆点Rect修改
- (CGRect)thumbRectForBounds:(CGRect)bounds trackRect:(CGRect)rect value:(float)value
{
    rect.origin.x = rect.origin.x - 3 ;
    rect.size.width = rect.size.width +6;
    return [super thumbRectForBounds:bounds trackRect:rect value:value];
}
~~~

### iOS获取View所在的控制器、导航控制器

~~~objective-c
//获取控制器
- (UIViewController*)viewController {
    for (UIView* next = [self superview]; next; next = next.superview) {
        UIResponder* nextResponder = [next nextResponder];
        if ([nextResponder isKindOfClass:[UINavigationController class]]) {
            return (UIViewController*)nextResponder;
        }
    }
    return nil;
}

//获取导航控制器
- (UINavigationController*)navigationController {
    for (UIView* next = [self superview]; next; next = next.superview) {
        UIResponder* nextResponder = [next nextResponder];
        if ([nextResponder isKindOfClass:[UINavigationController class]]) {
            return (UINavigationController*)nextResponder;
        }
    }
    return nil;
}
~~~

### UILabel 设置行间距

~~~objective-c
- (NSAttributedString *)getParagraphText:(NSString*)text LineSpacing:(CGFloat)spacing{
    if(text == nil || text.length == 0){
        return nil;
    }
    NSMutableAttributedString *attributedString = [[NSMutableAttributedString alloc]initWithString:text];;
    NSMutableParagraphStyle *paragraphStyle = [[NSMutableParagraphStyle alloc]init];
    [paragraphStyle setLineSpacing:spacing];
    [attributedString addAttribute:NSParagraphStyleAttributeName value:paragraphStyle range:NSMakeRange(0, text.length)];
    return attributedString;
}

//使用
self.titleLabel.attributedText = [self getParagraphText:infModel.title LineSpacing:7];
~~~

### UIView获取父控制器／父导航控制器

~~~objective-c
//创建UIView的分类
//获取当前控制器
- (UIViewController *)getCurrentVC {
    UIResponder *next = self.nextResponder;
    do {
        //判断响应者是否为视图控制器
        if ([next isKindOfClass:[UIViewController class]]) {
            return (UIViewController *)next;
        }
        next = next.nextResponder;
        
    } while (next != nil);
    return nil;
}

//获取导航控制器
- (UINavigationController *)getCurrentNavVC {
    UIResponder *next = self.nextResponder;
    do {
        //判断响应者是否为视图控制器
        if ([next isKindOfClass:[UINavigationController class]]) {
            return (UINavigationController *)next;
        }
        next = next.nextResponder;
        
    } while (next != nil);
    return nil;
}
~~~

### 拉伸图片

~~~objective-c
//拉伸图片
+ (UIImage *)resizedImageWithName:(NSString *)name
{
    return [self resizedImageWithName:name left:0.5 top:0.5];
}

+ (UIImage *)resizedImageWithName:(NSString *)name left:(CGFloat)left top:(CGFloat)top
{
    UIImage *image = [UIImage imageNamed:name];
    return [image stretchableImageWithLeftCapWidth:image.size.width * left topCapHeight:image.size.height * top];
}
~~~

### 判断当前控制器是push 还是present 

~~~objective-c
//方法一
- (IBAction)dismiss:(id)sender {
    if (self.presentingViewController) {
        [self dismissViewControllerAnimated:YES completion:nil];
    } else {
        [self.navigationController popViewControllerAnimated:YES];
    }
}

//方法二
- (IBAction)dismiss:(id)sender {
    if (self.navigationController.topViewController == self) {
        [self.navigationController popViewControllerAnimated:YES];
    } else {
        [self dismissViewControllerAnimated:YES completion:nil];
    }
}
~~~

### 导航栏全透明 、去除黑线

~~~objective-c
[self.navigationController.navigationBar setBackgroundImage:[UIImage new]
                                                  forBarMetrics:UIBarMetricsDefault];
    [self.navigationController.navigationBar setShadowImage:[UIImage new]];
~~~

### 获取文字字符数（汉字两个其他1个）

~~~objective-c
//返回字符个数 汉字两个字符
- (NSInteger)getStrLength:(NSString*)strtemp {
    NSStringEncoding enc = CFStringConvertEncodingToNSStringEncoding(kCFStringEncodingGB_18030_2000);
    NSData* da = [strtemp dataUsingEncoding:enc];
    return [da length];
}
~~~

### 判断iPhone尺寸

~~~objective-c

#define iPhone4Size (CGSizeEqualToSize(CGSizeMake(320, 480), [[UIScreen mainScreen] bounds].size))
#define iPhone5Size (CGSizeEqualToSize(CGSizeMake(320, 568), [[UIScreen mainScreen] bounds].size))
#define iPhone6Size (CGSizeEqualToSize(CGSizeMake(375, 667), [[UIScreen mainScreen] bounds].size))
#define iPhone6PlusSize (CGSizeEqualToSize(CGSizeMake(414,736), [[UIScreen mainScreen] bounds].size))
~~~

### 直播聊天列表的实现

~~~objective-c
- (void) addMessageWithStr:(NSString *)str {
    __weak typeof(self) weakSelf = self;
    if ([NSThread isMainThread]) {
        dispatch_async(dispatch_get_global_queue(0, 0), ^{
            [weakSelf insertChatView:str];
        });
    } else {
        [self insertChatView:str];
    }
}

- (void)insertChatView:(NSString *)str {
    dispatch_sync(dispatch_get_main_queue(), ^{
        if (self.array.count > 50) {
            
            for (int i = 0; i< 20; i++) {
                [_array removeObjectAtIndex:i];
                NSIndexPath *indexPath = [NSIndexPath indexPathForRow:i inSection:0];
                [self.tableView beginUpdates];
                [_tableView deleteRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationNone];
                [self.tableView endUpdates];
                
                NSInteger row = self.array.count>0 ? (self.array.count -1): 0 ;
                NSIndexPath *inde = [NSIndexPath indexPathForRow:row inSection:0];
                [self.tableView scrollToRowAtIndexPath:[NSIndexPath indexPathForRow:inde.row inSection:0]
                                      atScrollPosition:UITableViewScrollPositionTop
                                              animated:NO];
            }
        }
        [_array addObject:str];
        NSInteger row = self.array.count>0 ? (self.array.count -1): 0 ;
        NSIndexPath *indexPath = [NSIndexPath indexPathForRow:row inSection:0];
        [self.tableView beginUpdates];
        [self.tableView insertRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationNone];
        [self.tableView endUpdates];
        if (indexPath.row >0) {
            [self.tableView scrollToRowAtIndexPath:[NSIndexPath indexPathForRow:indexPath.row inSection:0]
                                  atScrollPosition:UITableViewScrollPositionBottom
                                          animated:YES];
        }
        
    });

}

~~~

### 全屏幕拖动返回简单实现

~~~objective-c
BaseNavigationController<UIGestureRecognizerDelegate>
//viewDidLoad中添加如下代码
id target = self.interactivePopGestureRecognizer.delegate;
    UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] initWithTarget:target action:@selector(handleNavigationTransition:)];
    [self.view addGestureRecognizer:pan];
    self.interactivePopGestureRecognizer.enabled = NO;
~~~

### 导航栏隐藏显示

~~~objective-c
- (void)viewWillAppear:(BOOL)animated {
    [self.navigationController setNavigationBarHidden:NO animated:animated];
    [super viewWillAppear:animated];
}

- (void)viewWillDisappear:(BOOL)animated {
    [self.navigationController setNavigationBarHidden:YES animated:animated];
    [super viewWillDisappear:animated];
    
}
~~~

### iOS TableView reloadData闪动的问题

~~~objective-c
解决方法：

//在iOS 11 Self-Sizing自动打开后，contentSize和contentOffset都可能发生改变。可以通过以下方式禁用

self.estimatedRowHeight = 0;
self.estimatedSectionHeaderHeight = 0;
self.estimatedSectionFooterHeight = 0;
~~~

### 避免父view视图手势影响

~~~objective-c
//设置手势代理
tap.delegate = self;

//实现代理方法
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch {
    if ([touch.view isDescendantOfView:self.alertView]) {
        return NO;
    }
    return YES;
}
~~~

### Scrollview嵌套 tableview

~~~objective-c
//scrollview contentSize的处理
- (void)viewDidLayoutSubviews {
    [super viewDidLayoutSubviews];
    
    [self.scrollView layoutIfNeeded];
    [self.view layoutIfNeeded];
    self.scrollView.contentSize = CGSizeMake(0, self.rewardView.bottom + 80 * 5 + 100 +20);
}
~~~

### presentViewController后view 透明效果设置

~~~objective-c
if ([[[UIDevice currentDevice] systemVersion] floatValue]>=8.0) {
    VC.modalPresentationStyle = UIModalPresentationOverCurrentContext;
}else{
    VC.modalPresentationStyle = UIModalPresentationCurrentContext;
}
[controller presentViewController:vC animated:NO completion:nil];
~~~

### AlertView动画效果

~~~objective-c
- (void)animationAlert:(UIView *)view
{
    CAKeyframeAnimation *popAnimation = [CAKeyframeAnimation animationWithKeyPath:@"transform"];
    popAnimation.duration = 0.4;
    popAnimation.removedOnCompletion = NO;
    popAnimation.fillMode = kCAFillModeForwards;
    popAnimation.values = @[[NSValue valueWithCATransform3D:CATransform3DMakeScale(0.01f, 0.01f, 1.0f)],
                            [NSValue valueWithCATransform3D:CATransform3DMakeScale(1.1f, 1.1f, 1.0f)],
                            [NSValue valueWithCATransform3D:CATransform3DMakeScale(0.9f, 0.9f, 1.0f)],
                            [NSValue valueWithCATransform3D:CATransform3DIdentity]];
    popAnimation.keyTimes = @[@0.0f, @0.5f, @0.75f, @1.0f];
    popAnimation.timingFunctions = @[[CAMediaTimingFunction
                                      functionWithName:kCAMediaTimingFunctionEaseIn],
                                     [CAMediaTimingFunction
                                      functionWithName:kCAMediaTimingFunctionEaseIn],
                                     [CAMediaTimingFunction
                                      functionWithName:kCAMediaTimingFunctionEaseIn]];
    
    [view.layer addAnimation:popAnimation forKey:nil];
    
}
~~~



