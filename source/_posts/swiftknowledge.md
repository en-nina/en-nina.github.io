---
title: swiftknowledge
date: 2019-05-08 13:21:42
categories: "iOS"
toc: true
tags: Swift
---

### #### swift裁剪图片模糊处理

~~~swift
extension UIImage {
    // 截取部分图片
    func imageAtRect(rect: CGRect) -> UIImage{
            var rect = rect
            rect.origin.x *= self.scale
            rect.origin.y *= self.scale
            rect.size.width *= self.scale
            rect.size.height *= self.scale
            let imageRef = self.cgImage!.cropping(to: rect)
            let image = UIImage(cgImage: imageRef!, scale: self.scale, orientation: self.imageOrientation)
            return image
    }
    
}
~~~



