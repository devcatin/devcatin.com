---
layout: post
title: "UIDynamic(捕捉行为)"
date: 2015-02-25 16:39:53 +0800
comments: true
categories: ios开发拓展篇

---

###一.简介
---
可以让物体迅速冲倒某个位置(捕捉位置)，捕捉到位置之后会带有一定的震动：

<!--more-->

UISnapBehavior的初始化

```ruby
- (instancetype)initWithItem:(id <UIDynamicItem>)item snapToPoint:(CGPoint)point;
```

UISnapBehavior常见属性

```
@property (nonatomic, assign) CGFloat damping;
```

用于减幅、减震（取值范围是0.0 ~ 1.0，值越大，震动幅度越小）

UISnapBehavior使用注意:

　　如果要进行连续的捕捉行为，需要先把前面的捕捉行为从物理仿真器中移除
　　
　　
###二.代码说明
---
在storyboard中放一个view控件，作为演示用的仿真元素。

代码如下：

```
//
//  ViewController.m
//  13-捕捉行为
//
//  Created by Grant on 15-2-25.
//  Copyright (c) 2014年 BG. All rights reserved.
//

#import "ViewController.h"

@interface ViewController ()

@property (weak, nonatomic) IBOutlet UIView *blueView;
@property (strong, nonatomic)UIDynamicAnimator *animator;

@end
 
@implementation ViewController
 
- (UIDynamicAnimator *)animator
{
    if (_animator == nil) 
    {
        //创建物理仿真器，设置仿真范围，ReferenceView为参照视图
        _animator = [[UIDynamicAnimator alloc]initWithReferenceView:self.view];
    }
    return _animator;
}

- (void)viewDidLoad
{
	[super viewDidLoad];
}

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    //获取一个触摸点
    UITouch *touch = [touches anyObject];
    CGPoint point = [touch locationInView:touch.view];
    
    //1.创建捕捉行为
    //需要传入两个参数：一个物理仿真元素，一个捕捉点
    UISnapBehavior *snap = [[UISnapBehavior alloc] initWithItem:self.blueView snapToPoint:point];
    
    //设置防震系数（0~1，数值越大，震动的幅度越小）
    snap.damping = arc4random_uniform(10) / 10.0;
    
    //2.执行捕捉行为
    //注意：这个控件只能用在一个仿真行为上，如果要拥有持续的仿真行为，那么需要把之前的所有仿真行为删除
    //删除之前的所有仿真行为
    [self.animator removeAllBehaviors];
    [self.animator addBehavior:snap];
}

@end
```
 小结：我们可以在做项目时加上以上的用户交互效果，会给用户不一样的体验。