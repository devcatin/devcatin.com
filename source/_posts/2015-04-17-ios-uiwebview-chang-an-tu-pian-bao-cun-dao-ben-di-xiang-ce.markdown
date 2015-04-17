---
layout: post
title: "IOS UIWebView 长按图片保存到本地相册"
date: 2015-01-19 13:51:50 +0800
comments: true
categories: ios探索

---

我们所要解决的问题如题目所示：ios中，长按Webview中的图片，将图片保存到本地相册。

<!--more-->

解决方案：对load的html网页，执行js注入，通过在webview中执行js代码，来响应点击事件，通过js代码来模拟长按事件。发现图片的位置，获得图片的url链接，通过此链接获得图片，将此图片保存到本地相册。

js注入代码：

```
static NSString* const kTouchJavaScriptString=
@"document.ontouchstart=function(event){\
x=event.targetTouches[0].clientX;\
y=event.targetTouches[0].clientY;\
document.location=\"myweb:touch:start:\"+x+\":\"+y;};\
document.ontouchmove=function(event){\
x=event.targetTouches[0].clientX;\
y=event.targetTouches[0].clientY;\
document.location=\"myweb:touch:move:\"+x+\":\"+y;};\
document.ontouchcancel=function(event){\
document.location=\"myweb:touch:cancel\";};\
document.ontouchend=function(event){\
document.location=\"myweb:touch:end\";};";
```
在开始之前先要定义几个宏

//#define GESTURE_STATE_START @"start"

//#define GESTURE_STATE_MOVE  @"move"

//define  GESTURE_STATE_END   @"end"

//define SNS_IMAGE_HINT_SAVE_FAILE @"Failure"

//define SNS_IMAGE_HINT_SAVE_SUCCE @"Sucess"

@property (strong, nonatomic) NSString *gesState

@property (strong, nonatomic) NSString *imgURL

@property (strong, nonatomic) NSTimer  *timer

js执行代码：

```ruby
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)_request navigationType:(UIWebViewNavigationType)navigationType {
    NSString *requestString = ［_request URL] absoluteString];
    NSArray *components = [requestString componentsSeparatedByString:@":"];
    if ([components count] > 1 && [(NSString *)[components objectAtIndex:0]
                                   isEqualToString:@"myweb"]) {
        if([(NSString *)[components objectAtIndex:1] isEqualToString:@"touch"])
        {
            //NSLog(@"you are touching!");
            //NSTimeInterval delaytime = Delaytime;
            if ([(NSString *)[components objectAtIndex:2] isEqualToString:@"start"]){
                 /*@需延时判断是否响应页面内的js*/
                _gesState = GESTURE_STATE_START;
                NSLog(@"touch start!");
                float ptX = ［components objectAtIndex:3]floatValue];
                float ptY = ［components objectAtIndex:4]floatValue];
                NSLog(@"touch point (%f, %f)", ptX, ptY);
                NSString *js = [NSString stringWithFormat:@"document.elementFromPoint(%f, %f).tagName", ptX, ptY];
                NSString * tagName = [self.getWebView stringByEvaluatingJavaScriptFromString:js];
                _imgURL = nil;
                if ([tagName isEqualToString:@"IMG"]) {
                    _imgURL = [NSString stringWithFormat:@"document.elementFromPoint(%f, %f).src", ptX, ptY];
                }
                if (_imgURL) {
                    _timer = [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(handleLongTouch) userInfo:nil repeats:NO];
                }
            }
            else if ([(NSString *)[components objectAtIndex:2] isEqualToString:@"move"])
            {
                //**如果touch动作是滑动，则取消hanleLongTouch动作**//
                _gesState = GESTURE_STATE_MOVE;
                NSLog(@"you are move");
                }
            }
            else if ([(NSString*)[components objectAtIndex:2]isEqualToString:@"end"]) {
                [_timer invalidate];
                _timer = nil;
                _gesState = GESTURE_STATE_END;
                NSLog(@"touch end");
            }
        }
        return NO;
    }
    return YES;
}
```

如果点击的是图片，并且按住的时间超过1s，执行handleLongTouch函数，处理图片的保存操作。

```

- (void)handleLongTouch {
    NSLog(@"%@", _imgURL);
    if (_imgURL && _gesState == GESTURE_STATE_START) {
        UIActionSheet* sheet = ［UIActionSheet alloc]initWithTitle:nil delegate:self cancelButtonTitle:@"取消" destructiveButtonTitle:nil otherButtonTitles:@"保存图片", nil];
        sheet.cancelButtonIndex = sheet.numberOfButtons - 1;
        [sheet showInView:[UIApplication sharedApplication].keyWindow];
    }
}
```
将图片保存到相册方法:

```

- (void)actionSheet:(UIActionSheet *)actionSheet clickedButtonAtIndex:(NSInteger)buttonIndex {
    if (actionSheet.numberOfButtons - 1 == buttonIndex) {
        return;
    }
    NSString* title = [actionSheet buttonTitleAtIndex:buttonIndex];
    if ([title isEqualToString:@"保存图片"]) {
        if (_imgURL) {
            NSLog(@"imgurl = %@", _imgURL);
        }
        NSString *urlToSave = [self.getWebView stringByEvaluatingJavaScriptFromString:_imgURL];
        NSLog(@"image url=%@", urlToSave);
        NSData* data = [NSData dataWithContentsOfURL:[NSURL URLWithString:urlToSave］;
        UIImage* image = [UIImage imageWithData:data];
        //UIImageWriteToSavedPhotosAlbum(image, nil, nil,nil);
        UIImageWriteToSavedPhotosAlbum(image, self, @selector(image:didFinishSavingWithError:contextInfo:), nil);
    }
}
```

保存成功或失败提示信息:

```

- (void)image:(UIImage *)image didFinishSavingWithError:(NSError*)error contextInfo:(void*)contextInfo
{
    if (error){
        NSLog(@"Error");
        [self showAlert:SNS_IMAGE_HINT_SAVE_FAILE];
    }else {
        NSLog(@"OK");
        [self showAlert:SNS_IMAGE_HINT_SAVE_SUCCE];
    }
}
```
总结：这篇文章主要是体现objecttive-c和js的交互。通过这篇文章，我们可以将app中UIWebView加载的html页面上的图片保存到自己的相册中。具体Demo就不提供了。