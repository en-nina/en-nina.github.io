---
title: iOS block、NSString之copy 探究
date: 2014-06-13 11:13:19
categories: "iOS"
toc: false
tags:
- Objective-C
description: 一探究竟
---

### block为啥用copy修饰

~~~objective-c
/**
  通俗理解, MRC下不使用copy修饰, block存储在栈区, 作用域结束, block这个局部变量被销毁, 而一个 strong指针还指向了这个被回收的地址.所以使用copy,把block拷贝到堆区, 并且指向堆区的block. 但是在ARC下无论copy,还是stong都无所谓, 因为系统已经在创建block的时候, 已经拷贝到堆区了.

   在 Objective-C 语言中，一共有 3 种类型的 block：
  __NSGlobalBlock__ 全局的静态 block，不会访问外部局部变量。
  __NSStackBlock__  保存在栈中的 block，访问外部局部变量，当函数返回时会被销毁。
  __NSMallocBlock__ 保存在堆中的 block，当引用计数为 0 时会被销毁。

*/

@interface Test ()  
@property (nonatomic, copy) void(^block)(); 
@end  
  
@implementation Test  
  
- (void)viewDidLoad {  
    [super viewDidLoad]; 
  
    //1 __NSGlobalBlock__  全局block   存储在代码区（存储方法或者函数）  
    void(^block1)() = ^() {  
        NSLog(@"一探究竟block1");  
    };  
    NSLog(@"block1 %@",block1);  
      
      
    //2 __NSStackBlock__  栈block  存储在栈区  
    //block内部访问外部变量   
    int n = 5;  
    void(^block2)() = ^() {  
        NSLog(@"一探究竟block2 %d", n);  
    };  
    NSLog(@"block2 %@", block2);  
      
      
      
      
     //3 __NSMallocBlock__  堆block 存储在堆区  对栈block做一次copy操作  
    void(^block3)() = ^() {  
        NSLog(@"一探究竟 block3%d", n);  
    };  
    NSLog(@"block3 %@", [block3 copy]);  
      
      
      
    /* 
       由以上可以看出当block没有访问外界的变量时,是存储在代码区,  
       当block访问外界变量时时存储在栈区, 
       而此时的block出了作用域就会被释放 
       以下示例: 
     */  
    [self test];
  	//当此代码结束时,test函数中的所有存储在栈区的变量都会被系统释放, 因此如果属性的block是用assign修饰时  当再次访问时就会出现野指针访问. 
  
    self.block();  
  
      
}  
  
- (void)test {  
    int n = 5;  
    [self setBlock:^{  
        NSLog(@"block %d",n);  
    }];  
     NSLog(@"test--%@",self.block);   
} 
~~~



### NSString 为啥用copy 修饰

~~~objective-c
/**
	对于NSString类型，使用copy修饰，不会改变它原有的类型，strong会指向引用的对象，有可能改变属性的值，所以copy能增强NSString的健壮性  同理 NSArray  NSDictionary
	
	当原字符串是NSMutableString类型时，strong特性对象只是增加了原字符串的引用计数，但是copy特性对象则是对原字符串进行了深拷贝，创建了一个新对象，并且指向了这个新对象。此时，copy特性对象是NSString类型的不可变的,strong特性对象是NSMutableString类型的可变的
*/

@interface ViewController ()

@property (nonatomic, strong)   NSString *str;
@property (nonatomic, copy)     NSString *strCopy;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    NSString *str = [NSString stringWithFormat:@"%@",@"a"];
    NSLog(@"str内存地址： %p",str); //0xa000000000000611
    self.str = str;
    
    NSLog(@"self.str 内存地址： %p",self.str);//0xa000000000000611   浅拷贝
    
    self.strCopy = str;
    
    NSLog(@"self.strCopy内存地址： %p",self.strCopy);//0xa000000000000611  浅拷贝
    
    NSMutableString *mutStr = [NSMutableString stringWithFormat:@"%@",@"xxx"];
    
    NSLog(@"mutStr内存地址： %p",mutStr);  //0x60800007d800
    
    self.str = mutStr;
    
    NSLog(@"str == %@ \n mutStr = %@",self.str,mutStr);
    [mutStr appendString:@"--------"];
    NSLog(@"str == %@ \n mutStr = %@",self.str,mutStr); //str值发生改变
    
    NSLog(@"self.str内存地址： %p",self.str);//0x60800007d800 strong指向同一个地址
    
    self.strCopy = mutStr;
    
    NSLog(@"strCopy == %@ \n mutStr = %@",self.strCopy,mutStr);
    [mutStr appendString:@"--------"];
    NSLog(@"strCopy == %@ \n mutStr = %@",self.strCopy,mutStr); //strCopy值不会发生改变
    
    NSLog(@"self.strCopy内存地址： %p",self.strCopy);//0xa000000007878783

}

@end

~~~

