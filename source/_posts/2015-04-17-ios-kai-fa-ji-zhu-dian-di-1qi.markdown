---
layout: post
title: "IOS 开发技术点第1期"
date: 2015-02-04 14:09:48 +0800
comments: true
categories: 问题集
---

以下是我工作中遇到的问题以及解决方案：

<!--more-->

####1.ios如何将文字转换成拼音
---

```ruby
- (void)convertWithChinese:(NSString *)text
{
    if (text.length > 0) {
        NSMutableString *ms = [[NSMutableString alloc] initWithString:text];
        if (CFStringTransform((__bridge CFMutableStringRef)ms, 0, kCFStringTransformMandarinLatin, NO)) {
            NSLog(@"pinyin: %@", ms);
        }
        if (CFStringTransform((__bridge CFMutableStringRef)ms, 0, kCFStringTransformStripDiacritics, NO)) {
            NSLog(@"pinyin: %@", ms);
        }
        pinyinLab.text = [ms copy];
    }
}
```

####2.ios开发之UIWebView
---

1).自动滑动到顶部

```
- (void)automaticSlidingToTheTop
{
	if (_mWebView.subviews.count > 0) 
	{
		UIScrollView *scrollView = [_mWebView.subviews objectAtIndex:0];
	    [scrollView setContentOffset:CGPointMake(0, 0) animated:YES];
	}
}
```
2).限制UIWebView左右滑动

```
- (void)limitSlidingAround
{
	for (id subview in _mWebView.subviews)
	{
	    if ([[subview class] isSubclassOfClass:[UIScrollView class]])      
	    {
	        ((UIScrollView *)subview).bounces = NO;
	    }
	}
}
```
####3.ios开发之AFNetWorking同步和异步
---

1)同步

```
- (void)requestFromServer
{
	NSString *url = [NSString stringWithFormat:@"%@/s/account/find-user/",ServerBaseURL];
	NSMutableDictionary *requestParms = [[NSMutableDictionary alloc] init];
	[requestParms setObject:userID forKey:@"userId"];
	AFJSONRequestSerializer *requestSerializer = [AFJSONRequestSerializer serializer];
	NSMutableURLRequest *request = [requestSerializer requestWithMethod:@"POST" URLString:url parameters:requestParms error:nil];
	AFHTTPRequestOperation *requestOperation = [[AFHTTPRequestOperation alloc] initWithRequest:request];
	AFHTTPResponseSerializer *responseSerializer = [AFJSONResponseSerializer serializer];
	[requestOperation setResponseSerializer:responseSerializer];
	[requestOperation start];
	[requestOperation waitUntilFinished];
	NSDictionary *userInfo = [[requestOperation responseObject] objectForKey:@"user”];
}
```

2).异步

```
- (void)requestFromServer
{
	NSString *url = [NSString stringWithFormat:@"%@/s/account/signin/", ServerBaseURL];
	NSMutableDictionary *requestParms = [[NSMutableDictionary alloc] init];
	//设置参数
	[requestParms setObject:loginName forKey:@"loginname"];
	[httpManager POST:url parameters:requestParms
    success:^(AFHTTPRequestOperation *operation, id responseObject) 
    {
		//成功
	} failure:^(AFHTTPRequestOperation *operation, NSError *error) {
		//失败
	}];
}
```
####4.iphone 屏幕尺寸:
---

iphone 4/4s   : 320x480、640x960

iphone 5/5s   : 320x568、640x1136

iphone 6      : 375x667、750x1334

iphone 6 plus : 414x736、1242x2208

icon:29,29@2x,29@3x,40,40@2x,40@3x,60@2x,60@3x,76,76@2x(iphone和ipad)

####5.ios中arc和非arc模式混用
---

Xcode 项目中我们可以使用 ARC 和非 ARC 的混合模式。
如果你的项目使用的非 ARC 模式，则为 ARC 模式的代码文件加入 -fobjc-arc 标签。
如果你的项目使用的是 ARC 模式，则为非 ARC 模式的代码文件加入 -fno-objc-arc 标签。
添加标签的方法：
	1).	打开：你的target -> Build Phases -> Compile Sources.
	2).	双击对应的 *.m 文件
	3).	在弹出窗口中输入上面提到的标签 -fobjc-arc / -fno-objc-arc
	4).	点击 done 保存

####6.ios开发之计时器
---

1).使用GCD实现倒计时

```
- (void)countDownTimer:(id)btn
{
    __block int timeout = 60;//倒计时时间
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_source_t _timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
    dispatch_source_set_timer(_timer, dispatch_walltime(NULL, 0), 1.0 * NSEC_PER_SEC, 0);//每秒执行
    dispatch_source_set_event_handler(_timer, ^
    {
        //倒计时结束，关闭
        if (timeout <= 0)
        {
            dispatch_source_cancel(_timer);
            dispatch_release(_timer);
            dispatch_async(dispatch_get_main_queue(), ^
            {
                //根据自己的需要，设置界面的按钮显示
                [btn setTitle:@"获取验证码" forState:UIControlStateNormal];
            });
        }
        else
        {
            NSString *strTime = [NSString stringWithFormat:@"%d秒",timeout];            
            dispatch_async(dispatch_get_main_queue(), ^
            {
                //根据自己的需要，设置界面的按钮显示                
                [btn setTitle:strTime forState:UIControlStateNormal];            
                });            
                timeout --;
             }
    });    
    dispatch_resume(_timer);
}
```
2).使用NSTimer来实现

    主要使用的是NSTimer的scheduledTimerWithTimeInterval方法来每1秒执行一次timeFireMethod函数，timeFireMethod进行倒计时的一些操作，完成时把timer给invalidate掉就ok了，代码如下：

```
- (void)timer
{
	secondsCountDown = 60;//60秒倒计时
	countDownTimer = [NSTimer scheduledTimerWithTimeInterval:target:self selector:@selector(timeFireMethod) userInfo:nil repeats:YES];  
	[self timeFireMethod]
} 
- (void)timeFireMethod
{  
    secondsCountDown--;  
    if(secondsCountDown == 0)
    {  
    	[countDownTimer invalidate];  
    }  
}
```

小结:针对以上的这点小技术点，有些让我纠结了很长时间，现在好了，都给解决了，所以我想这些对大家来说也是需要知道的。