---
title: iphoneX系列适配用的宏定义
date: 2018-10-19 09:58:26
categories: iOS
tags: iOS
---

去年适配的iPhoneX 的分辨率：2436 * 1125     ||     pt: 812 * 375
iPhoneXr的分辨率：1792 * 828     ||    pt:  896 * 414
iPhoneXs 的分辨率： 2436 * 1125   ||   pt: 812 * 375
iPhoneXs Max 的分辨率：2688 * 1242   ||    pt: 896 * 414

导航栏和tabBar的高度是一样的，所以需要在原来判断iPhoneX的基础上增加两种机型.

```objective-c
//判断是否是ipad
#define isPad ([[UIDevice currentDevice] userInterfaceIdiom] == UIUserInterfaceIdiomPad)
//判断iPhone4系列
#define kiPhone4 ([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(640, 960), [[UIScreen mainScreen] currentMode].size) && !isPad : NO)
//判断iPhone5系列
#define kiPhone5 ([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(640, 1136), [[UIScreen mainScreen] currentMode].size) && !isPad : NO)
//判断iPhone6系列
#define kiPhone6 ([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(750, 1334), [[UIScreen mainScreen] currentMode].size) && !isPad : NO)
//判断iphone6+系列
#define kiPhone6Plus ([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(1242, 2208), [[UIScreen mainScreen] currentMode].size) && !isPad : NO)
//判断iPhoneX
#define IS_IPHONE_X ([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(1125, 2436), [[UIScreen mainScreen] currentMode].size) && !isPad : NO)
//判断iPHoneXr
#define IS_IPHONE_Xr ([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(828, 1792), [[UIScreen mainScreen] currentMode].size) && !isPad : NO)
//判断iPhoneXs
#define IS_IPHONE_Xs ([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(1125, 2436), [[UIScreen mainScreen] currentMode].size) && !isPad : NO)
//判断iPhoneXs Max
#define IS_IPHONE_Xs_Max ([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(1242, 2688), [[UIScreen mainScreen] currentMode].size) && !isPad : NO)

```

navBar和tabBar的判断

```objective-c
//iPhoneX系列
#define Height_StatusBar ((IS_IPHONE_X==YES || IS_IPHONE_Xr ==YES || IS_IPHONE_Xs== YES || IS_IPHONE_Xs_Max== YES) ? 44.0 : 20.0)
#define Height_NavBar ((IS_IPHONE_X==YES || IS_IPHONE_Xr ==YES || IS_IPHONE_Xs== YES || IS_IPHONE_Xs_Max== YES) ? 88.0 : 64.0)
#define Height_TabBar ((IS_IPHONE_X==YES || IS_IPHONE_Xr ==YES || IS_IPHONE_Xs== YES || IS_IPHONE_Xs_Max== YES) ? 83.0 : 49.0)

```

横屏状态下还有变化

~~~objective-c
#define CC_IsPortrait ([[UIApplication sharedApplication] statusBarOrientation] == UIInterfaceOrientationPortrait || [[UIApplication sharedApplication] statusBarOrientation] == UIInterfaceOrientationPortraitUpsideDown)

//iphonex 安全区域
#define CC_SafeEdgeInsets  (CC_IsPortrait ? UIEdgeInsetsMake(44, 0, 34, 0) : UIEdgeInsetsMake(0, 44, 21, 44))

~~~



**Swift**

~~~swift
/// 屏幕宽度
let LS_SCREEN_WIDTH = UIScreen.main.bounds.width

/// 屏幕高度
let LS_SCREEN_HEIGHT = UIScreen.main.bounds.height

/// 屏幕bounds
let LS_SCREEN_BOUNDS = UIScreen.main.bounds

let LS_WINDOW = UIApplication.shared.delegate?.window

/// 判断是不是iPad
let LS_IS_IPAD = UIDevice.current.userInterfaceIdiom == UIUserInterfaceIdiom.pad


/// 判断是不是iPhone5
let LS_IS_IPHONE_5 = UIScreen.instancesRespond(to: #selector(getter: UIScreen.currentMode)) ? __CGSizeEqualToSize(CGSize(width: 640, height: 1136), UIScreen.main.currentMode?.size ?? CGSize.zero) && !LS_IS_IPAD : false

/// 判断是不是iPhone6
let LS_IS_IPHONE_6 = UIScreen.instancesRespond(to: #selector(getter: UIScreen.currentMode)) ? __CGSizeEqualToSize(CGSize(width: 750, height: 1334), UIScreen.main.currentMode?.size ?? CGSize.zero) && !LS_IS_IPAD : false

/// 判断是不是iPhone6
let LS_IS_IPHONE_6P = UIScreen.instancesRespond(to: #selector(getter: UIScreen.currentMode)) ? __CGSizeEqualToSize(CGSize(width: 1242, height: 2208), UIScreen.main.currentMode?.size ?? CGSize.zero) && !LS_IS_IPAD : false

let LS_IS_IPHONE_6P_SCALE = UIScreen.instancesRespond(to: #selector(getter: UIScreen.currentMode)) ? __CGSizeEqualToSize(CGSize(width: 1125, height: 2001), UIScreen.main.currentMode?.size ?? CGSize.zero) && !LS_IS_IPAD : false

/// 判断是不是iPhoneX XS
let LS_IS_IPHONEX = UIScreen.instancesRespond(to: #selector(getter: UIScreen.currentMode)) ? __CGSizeEqualToSize(CGSize(width: 1125, height: 2436), UIScreen.main.currentMode?.size ?? CGSize.zero) && !LS_IS_IPAD : false

let LS_IS_IPHONEXR = UIScreen.instancesRespond(to: #selector(getter: UIScreen.currentMode)) ? __CGSizeEqualToSize(CGSize(width: 828, height: 1792), UIScreen.main.currentMode?.size ?? CGSize.zero) && !LS_IS_IPAD : false

let LS_IS_IPHONEXS_MAX = UIScreen.instancesRespond(to: #selector(getter: UIScreen.currentMode)) ? __CGSizeEqualToSize(CGSize(width: 1242, height: 2688), UIScreen.main.currentMode?.size ?? CGSize.zero) && !LS_IS_IPAD : false

let LS_IS_IPHONEX_SERIES = (LS_IS_IPHONEX || LS_IS_IPHONEXR || LS_IS_IPHONEXS_MAX)


let kNavHeight = CGFloat(LS_IS_IPHONEX_SERIES ? 88 : 64)
let kTabbarHeight = CGFloat(LS_IS_IPHONEX_SERIES ? 83 : 49)
let kSafeAreaBottomHeight = CGFloat(LS_IS_IPHONEX_SERIES ? 34 : 0)
let kStatusBarHeight = UIApplication.shared.statusBarFrame.size.height
~~~



第二种

~~~swift
public extension UIDevice {
    var deviceName: String {
        
        var systemInfo = utsname()
        uname(&systemInfo)
        
        let machineMirror = Mirror(reflecting: systemInfo.machine)
        let identifier = machineMirror.children.reduce("") {identifier, element  in
            guard let value = element.value as? Int8, value != 0 else {
                return identifier
            }
            return identifier + String(UnicodeScalar(UInt8(value)))
        }
        
        switch identifier {
        case "iPhone5,1",
             "iPhone5,2",
             "iPhone5,3",
             "iPhone5,4",
             "iPhone6,1",
             "iPhone6,2":
            return "iPhone5" //5系列
            
        case "iPhone7,2",
             "iPhone8,1",
             "iPhone9,1",
             "iPhone9,3",
             "iPhone10,1":
            return "iPhone678" // 6、7、8
            
            
        case "iPhone7,1",
             "iPhone8,2",
             "iPhone9,2",
             "iPhone9,4",
             "iPhone10,2",
             "iPhone10,5":
            return "iPhone678Plus"
            
        case "iPhone10,3",
             "iPhone10,6",
             "iPhone11,8",
             "iPhone11,2",
             "iPhone11,6":
            return "iPhoneXSeries"
            
        case "iPhone8,4":  return "iPhoneSE"
            
        default:  return identifier
        }
    }
    
    
    var isPhone5Series: Bool {
        return deviceName == "iPhone5"
    }
    
    var isPhone6Series: Bool {
        return deviceName == "iPhone678"
    }
    
    var isPhone6PSeries: Bool {
        return deviceName == "iPhone678Plus"
    }
    
    var isPhoneXSeries: Bool {
        return deviceName == "iPhoneXSeries"
    }
    
    
}

~~~



