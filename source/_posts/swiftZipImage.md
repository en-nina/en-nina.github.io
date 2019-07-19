---
title: swiftZipImage
date: 2019-07-17 19:06:59
tags: Swift
---

# iOS图片压缩上传

## 需求

很多时候我们上传图片经常遇到一些问题，要不就是图片质量变差，要不就是图片太大等等问题。这里，我找到了一个算是目前比较符合需求的解决方案。在原有基础上增加了动态压缩系数，改写成Swift版本，最底下贴出OC版本。

## 实现思路

先调整分辨率，分辨率可以自己设定一个值，大于的就缩小到这分辨率，小余的就保持原本分辨率。然后再根据图片最终大小来设置压缩比，比如传入maxSize = 30KB，最终计算大概这个大小的压缩比。基本上最终出来的图片数据根据当前分辨率能保持差不多的大小同时不至于太模糊，跟微信，微博最终效果应该是差不多的，代码仍然有待优化！

## 实现代码

### OC 版本（推荐）

基于网友的要求，我把 OC 版本的代码也贴出来。

```objective-c
#pragma mark - 压缩图片到指定大小(单位KB)
+ (NSData *)resetSizeOfImageData:(UIImage *)sourceImage maxSize:(NSInteger)maxSize {
    //先判断当前质量是否满足要求，不满足再进行压缩
    __block NSData *finallImageData = UIImageJPEGRepresentation(sourceImage,1.0);
    NSUInteger sizeOrigin   = finallImageData.length;
    NSUInteger sizeOriginKB = sizeOrigin / 1000;
    
    if (sizeOriginKB <= maxSize) {
        return finallImageData;
    }
    
    //获取原图片宽高比
    CGFloat sourceImageAspectRatio = sourceImage.size.width/sourceImage.size.height;
    //先调整分辨率
    CGSize defaultSize = CGSizeMake(1024, 1024/sourceImageAspectRatio);
    UIImage *newImage = [self newSizeImage:defaultSize image:sourceImage];
    
    finallImageData = UIImageJPEGRepresentation(newImage,1.0);
    
    //保存压缩系数
    NSMutableArray *compressionQualityArr = [NSMutableArray array];
    CGFloat avg   = 1.0/250;
    CGFloat value = avg;
    for (int i = 250; i >= 1; i--) {
        value = i*avg;
        [compressionQualityArr addObject:@(value)];
    }
    
    /*
     调整大小
     说明：压缩系数数组compressionQualityArr是从大到小存储。
     */
    //思路：使用二分法搜索
    __block NSData *canCompressMinData = [NSData data];//当无法压缩到指定大小时，用于存储当前能够压缩到的最小值数据。
    [self halfFuntion:compressionQualityArr image:newImage sourceData:finallImageData maxSize:maxSize resultBlock:^(NSData *finallData, NSData *tempData) {
        finallImageData = finallData;
        canCompressMinData = tempData;
    }];
    //如果还是未能压缩到指定大小，则进行降分辨率
    while (finallImageData.length == 0) {
        //每次降100分辨率
        CGFloat reduceWidth = 100.0;
        CGFloat reduceHeight = 100.0/sourceImageAspectRatio;
        if (defaultSize.width-reduceWidth <= 0 || defaultSize.height-reduceHeight <= 0) {
            break;
        }
        defaultSize = CGSizeMake(defaultSize.width-reduceWidth, defaultSize.height-reduceHeight);
        UIImage *image = [self newSizeImage:defaultSize
                                      image:[UIImage imageWithData:UIImageJPEGRepresentation(newImage,[[compressionQualityArr lastObject] floatValue])]];
        [self halfFuntion:compressionQualityArr image:image sourceData:UIImageJPEGRepresentation(image,1.0) maxSize:maxSize resultBlock:^(NSData *finallData, NSData *tempData) {
            finallImageData = finallData;
            canCompressMinData = tempData;
        }];
    }
    //如果分辨率已经无法再降低，则直接使用能够压缩的那个最小值即可
    if (finallImageData.length==0) {
        finallImageData = canCompressMinData;
    }
    return finallImageData;
}

#pragma mark 调整图片分辨率/尺寸（等比例缩放）
///调整图片分辨率/尺寸（等比例缩放）
+ (UIImage *)newSizeImage:(CGSize)size image:(UIImage *)sourceImage {
    CGSize newSize = CGSizeMake(sourceImage.size.width, sourceImage.size.height);
    
    CGFloat tempHeight = newSize.height / size.height;
    CGFloat tempWidth = newSize.width / size.width;
    
    if (tempWidth > 1.0 && tempWidth > tempHeight) {
        newSize = CGSizeMake(sourceImage.size.width / tempWidth, sourceImage.size.height / tempWidth);
    } else if (tempHeight > 1.0 && tempWidth < tempHeight) {
        newSize = CGSizeMake(sourceImage.size.width / tempHeight, sourceImage.size.height / tempHeight);
    }
    
    //    UIGraphicsBeginImageContext(newSize);
    UIGraphicsBeginImageContextWithOptions(newSize, NO, 1);
    [sourceImage drawInRect:CGRectMake(0,0,newSize.width,newSize.height)];
    UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return newImage;
}

#pragma mark 二分法
///二分法，block回调中finallData长度不为零表示最终压缩到了指定的大小，如果为零则表示压缩不到指定大小。tempData表示当前能够压缩到的最小值。
+ (void)halfFuntion:(NSArray *)arr image:(UIImage *)image sourceData:(NSData *)finallImageData maxSize:(NSInteger)maxSize resultBlock:(void(^)(NSData *finallData, NSData *tempData))block {
    NSData *tempData = [NSData data];
    NSUInteger start = 0;
    NSUInteger end = arr.count - 1;
    NSUInteger index = 0;
    
    NSUInteger difference = NSIntegerMax;
    while(start <= end) {
        index = start + (end - start)/2;
        
        finallImageData = UIImageJPEGRepresentation(image,[arr[index] floatValue]);
        
        NSUInteger sizeOrigin = finallImageData.length;
        NSUInteger sizeOriginKB = sizeOrigin / 1000;
        NSLog(@"当前降到的质量：%ld", (unsigned long)sizeOriginKB);
        //        NSLog(@"\nstart：%zd\nend：%zd\nindex：%zd\n压缩系数：%lf", start, end, (unsigned long)index, [arr[index] floatValue]);
        
        if (sizeOriginKB > maxSize) {
            start = index + 1;
        } else if (sizeOriginKB < maxSize) {
            if (maxSize-sizeOriginKB < difference) {
                difference = maxSize-sizeOriginKB;
                tempData = finallImageData;
            }
            if (index<=0) {
                break;
            }
            end = index - 1;
        } else {
            break;
        }
    }
    NSData *d = [NSData data];
    if (tempData.length==0) {
        d = finallImageData;
    }
    if (block) {
        block(tempData, d);
    }
//    return tempData;
}
```

### Swift3.0版本二分法压缩模式（推荐）

```swift
// MARK: - 降低质量
    func resetSizeOfImageData(sourceImage: UIImage!, maxSize: Int) -> NSData {
        
        //先判断当前质量是否满足要求，不满足再进行压缩
        var finallImageData = UIImageJPEGRepresentation(sourceImage,1.0)
        let sizeOrigin      = finallImageData?.count
        let sizeOriginKB    = sizeOrigin! / 1024
        if sizeOriginKB <= maxSize {
            return finallImageData! as NSData
        }
        
        //获取原图片宽高比
        let sourceImageAspectRatio = sourceImage.size.width/sourceImage.size.height
        //先调整分辨率
        var defaultSize = CGSize(width: 1024, height: 1024/sourceImageAspectRatio)
        let newImage = self.newSizeImage(size: defaultSize, sourceImage: sourceImage)
        
        finallImageData = UIImageJPEGRepresentation(newImage,1.0);
        
        //保存压缩系数
        let compressionQualityArr = NSMutableArray()
        let avg = CGFloat(1.0/250)
        var value = avg
        
        var i = 250
        repeat {
            i -= 1
            value = CGFloat(i)*avg
            compressionQualityArr.add(value)
        } while i >= 1

        
        /*
         调整大小
         说明：压缩系数数组compressionQualityArr是从大到小存储。
         */
        //思路：使用二分法搜索
        finallImageData = self.halfFuntion(arr: compressionQualityArr.copy() as! [CGFloat], image: newImage, sourceData: finallImageData!, maxSize: maxSize)
        //如果还是未能压缩到指定大小，则进行降分辨率
        while finallImageData?.count == 0 {
            //每次降100分辨率
            let reduceWidth = 100.0
            let reduceHeight = 100.0/sourceImageAspectRatio
            if (defaultSize.width-CGFloat(reduceWidth)) <= 0 || (defaultSize.height-CGFloat(reduceHeight)) <= 0 {
                break
            }
            defaultSize = CGSize(width: (defaultSize.width-CGFloat(reduceWidth)), height: (defaultSize.height-CGFloat(reduceHeight)))
            let image = self.newSizeImage(size: defaultSize, sourceImage: UIImage.init(data: UIImageJPEGRepresentation(newImage, compressionQualityArr.lastObject as! CGFloat)!)!)
            finallImageData = self.halfFuntion(arr: compressionQualityArr.copy() as! [CGFloat], image: image, sourceData: UIImageJPEGRepresentation(image,1.0)!, maxSize: maxSize)
        }
        
        return finallImageData! as NSData
    }
    
    // MARK: - 调整图片分辨率/尺寸（等比例缩放）
    func newSizeImage(size: CGSize, sourceImage: UIImage) -> UIImage {
        var newSize = CGSize(width: sourceImage.size.width, height: sourceImage.size.height)
        let tempHeight = newSize.height / size.height
        let tempWidth = newSize.width / size.width
        
        if tempWidth > 1.0 && tempWidth > tempHeight {
            newSize = CGSize(width: sourceImage.size.width / tempWidth, height: sourceImage.size.height / tempWidth)
        } else if tempHeight > 1.0 && tempWidth < tempHeight {
            newSize = CGSize(width: sourceImage.size.width / tempHeight, height: sourceImage.size.height / tempHeight)
        }
        
        UIGraphicsBeginImageContext(newSize)
        sourceImage.draw(in: CGRect(x: 0, y: 0, width: newSize.width, height: newSize.height))
        let newImage = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        return newImage!
    }
    
    // MARK: - 二分法
    func halfFuntion(arr: [CGFloat], image: UIImage, sourceData finallImageData: Data, maxSize: Int) -> Data? {
        var tempFinallImageData = finallImageData
        
        var tempData = Data.init()
        var start = 0
        var end = arr.count - 1
        var index = 0
        
        var difference = Int.max
        while start <= end {
            index = start + (end - start)/2
            
            tempFinallImageData = UIImageJPEGRepresentation(image, arr[index])!
            
            let sizeOrigin = tempFinallImageData.count
            let sizeOriginKB = sizeOrigin / 1024
            
            print("当前降到的质量：\(sizeOriginKB)\n\(index)----\(arr[index])")
            
            if sizeOriginKB > maxSize {
                start = index + 1
            } else if sizeOriginKB < maxSize {
                if maxSize-sizeOriginKB < difference {
                    difference = maxSize-sizeOriginKB
                    tempData = tempFinallImageData
                }
                if index<=0 {
                    break
                }
                end = index - 1
            } else {
                break
            }
        }
        return tempData
    }
```

### ~~Swift3.0之前旧版本压缩模式（建议不用，性能太差）~~

```swift
// MARK: - 降低质量
    func resetSizeOfImageData(source_image: UIImage, maxSize: Int) -> NSData {
        //先调整分辨率
        var newSize = CGSize(width: source_image.size.width, height: source_image.size.height)
        
        let tempHeight = newSize.height / 1024
        let tempWidth  = newSize.width / 1024
        
        if tempWidth > 1.0 && tempWidth > tempHeight {
            newSize = CGSize(width: source_image.size.width / tempWidth, height: source_image.size.height / tempWidth)
        }
        else if tempHeight > 1.0 && tempWidth < tempHeight {
            newSize = CGSize(width: source_image.size.width / tempHeight, height: source_image.size.height / tempHeight)
        }
        
        UIGraphicsBeginImageContext(newSize)
        source_image.drawAsPatternInRect(CGRect(x: 0, y: 0, width: newSize.width, height: newSize.height))
        let newImage = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        
        //先判断当前质量是否满足要求，不满足再进行压缩
        var finallImageData = UIImageJPEGRepresentation(newImage,1.0)
        let sizeOrigin      = Int64((finallImageData?.length)!)
        let sizeOriginKB    = Int(sizeOrigin / 1024)
        if sizeOriginKB <= maxSize {
            return finallImageData!
        }
        
        //保存压缩系数
        let compressionQualityArr = NSMutableArray()
        let avg = CGFloat(1.0/250)
        var value = avg
        
        for var i = 250; i>=1; i-- {
            value = CGFloat(i)*avg
            compressionQualityArr.addObject(value)
        }
        
        //调整大小
        //说明：压缩系数数组compressionQualityArr是从大到小存储。
        //思路：折半计算，如果中间压缩系数仍然降不到目标值maxSize，则从后半部分开始寻找压缩系数；反之从前半部分寻找压缩系数
        finallImageData = UIImageJPEGRepresentation(newImage, CGFloat(compressionQualityArr[125] as! NSNumber))
        if Int(Int64((UIImageJPEGRepresentation(newImage, CGFloat(compressionQualityArr[125] as! NSNumber))?.length)!)/1024) > maxSize {
            //拿到最初的大小
            finallImageData = UIImageJPEGRepresentation(newImage, 1.0)
            //从后半部分开始
            for idx in 126..<250 {
                let value = compressionQualityArr[idx]
                let sizeOrigin   = Int64((finallImageData?.length)!)
                let sizeOriginKB = Int(sizeOrigin / 1024)
                print("当前降到的质量：\(sizeOriginKB)")
                if sizeOriginKB > maxSize {
                    print("\(idx)----\(value)")
                    finallImageData = UIImageJPEGRepresentation(newImage, CGFloat(value as! NSNumber))
                } else {
                    break
                }
            }
        } else {
            //拿到最初的大小
            finallImageData = UIImageJPEGRepresentation(newImage, 1.0)
            //从前半部分开始
            for idx in 0..<125 {
                let value = compressionQualityArr[idx]
                let sizeOrigin   = Int64((finallImageData?.length)!)
                let sizeOriginKB = Int(sizeOrigin / 1024)
                print("当前降到的质量：\(sizeOriginKB)")
                if sizeOriginKB > maxSize {
                    print("\(idx)----\(value)")
                    finallImageData = UIImageJPEGRepresentation(newImage, CGFloat(value as! NSNumber))
                } else {
                    break
                }
            }
        }
        return finallImageData!
    }
```

【更新日志】
2017年10月09日：修复了网友提出的二分法存在index为0时闪退问题。

2018年05月25日：将原本默认设置图片尺寸为1024*1024改成等比放大，同时降低分辨力也改成等比降低。