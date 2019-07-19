---
title: swift登录退出登录切换rootViewController完全销毁处理方案
date: 2019-07-19 16:01:46
tags: Swift
---



**present之后重新设置rootvc会出现的问题解决**

~~~swift
static func presentLogin() {
        let nav = UINavigationController.init(rootViewController: LSLoginViewController())
        
        if let vc = getTopVC() {
            var parentVC = vc.presentingViewController
            var bottomVC: UIViewController?
            while (parentVC != nil) {
                bottomVC = parentVC
                parentVC = parentVC?.presentingViewController
            }
            
            if bottomVC == nil {
                if let window = UIApplication.shared.delegate?.window {
                    window?.rootViewController  = nav
                }
            } else {
                bottomVC?.dismiss(animated: false, completion: {
                    if let window = UIApplication.shared.delegate?.window {
                        window?.rootViewController  = nav
                    }
                })
            }
        } else {
            if let window = UIApplication.shared.delegate?.window {
                window?.rootViewController  = nav
            }
        }
    }
    
    static func loginSuccess() {
        if let window = UIApplication.shared.delegate?.window {
            window?.rootViewController  = TabBarController()
        }       
    }
    
~~~



~~~swift
// MARK: - 查找顶层控制器、
// 获取顶层控制器 根据window
func getTopVC() -> (UIViewController?) {
    var window = UIApplication.shared.keyWindow
    //是否为当前显示的window
    if window?.windowLevel != UIWindow.Level.normal{
        let windows = UIApplication.shared.windows
        for  windowTemp in windows{
            if windowTemp.windowLevel == UIWindow.Level.normal{
                window = windowTemp
                break
            }
        }
    }
    let vc = window?.rootViewController
    return getTopVC(withCurrentVC: vc)
}
///根据控制器获取 顶层控制器
func getTopVC(withCurrentVC vc :UIViewController?) -> UIViewController? {
    if vc == nil {
        print("🌶： 找不到顶层控制器")
        return nil
    }
    if let presentVC = vc?.presentedViewController {
        //modal出来的 控制器
        return getTopVC(withCurrentVC: presentVC)
    }else if let tabVC = vc as? UITabBarController {
        // tabBar 的跟控制器
        if let selectVC = tabVC.selectedViewController {
            return getTopVC(withCurrentVC: selectVC)
        }
        return nil
    } else if let naiVC = vc as? UINavigationController {
        // 控制器是 nav
        return getTopVC(withCurrentVC:naiVC.visibleViewController)
    } else {
        // 返回顶控制器
        return vc
    }
}

~~~

