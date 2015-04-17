---
layout: post
title: "基于第三方微信授权登录的iOS代码分析"
date: 2014-12-17 13:14:31 +0800
comments: true
categories: ios探索

---

微信已经深入到每一个APP的缝隙，最常用的莫过分享和登录了，接下来就以代码的形式来展开微信登录的相关说明，至于原理级别的oauth2.0认证体系请参考微信开放平台的相关说明和图示 https://open.weixin.qq.com/

<!--more-->

##微信登录授权开发

>1.到微信开发平台注册相关APP，现在是等待审核成功后才能获取到对应的`key`和`secret`；获取成功后需要单独申请开通登录和支付接口，如图

![](https://raw.githubusercontent.com/zhangkt/zhangkt.github.com/master/images/blogimages/1411370441623372.png)

2.和QQ类似，需要填写`Url Schemes`，如demo中的wxd930ea5d5a258f4f ，然后引入相应framework；

3.在AppDelegate中注册和实现授权后的回调函数，代码如下：

```ruby
//向微信注册  
  [WXApi registerApp:kWXAPP_ID withDescription:@"weixin"];  
//授权后回调 WXApiDelegate  
-(void)onResp:(BaseReq *)resp  
{  
   /* 
    ErrCode ERR_OK = 0(用户同意) 
    ERR_AUTH_DENIED = -4（用户拒绝授权） 
    ERR_USER_CANCEL = -2（用户取消） 
    code    用户换取access_token的code，仅在ErrCode为0时有效 
    state   第三方程序发送时用来标识其请求的唯一性的标志，由第三方程序调用sendReq时传入，由微信终端回传，state字符串长度不能超过1K 
    lang    微信客户端当前语言 
    country 微信用户当前国家信息 
    */      
    SendAuthResp *aresp = (SendAuthResp *)resp;  
    if (aresp.errCode== 0) {  
        NSString *code = aresp.code;  
        NSDictionary *dic = @{@"code":code};  
    }  
}  
//和QQ,新浪并列回调句柄
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation  
{  
    return [TencentOAuth HandleOpenURL:url] ||  
    [WeiboSDK handleOpenURL:url delegate:self] ||  
    [WXApi handleOpenURL:url delegate:self];;  
}  
- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url  
{  
    return [TencentOAuth HandleOpenURL:url] ||  
    [WeiboSDK handleOpenURL:url delegate:self] ||  
    [WXApi handleOpenURL:url delegate:self];;  
}
```

4.微信登录授权比较复杂，相比QQ，新浪多了几步，简单说就是需要三步，第一步，获取`code`，这个用来获取token，第二步，就是带上code获取`token`，第三步，根据第二步获取的token和openid来获取用户的相关信息；

下面用代码来实现：

第一步：code

```
- (IBAction)weixinLogin:(id)sender  
{  
    [self sendAuthRequest];  
}  
-(void)sendAuthRequest  
{  
    SendAuthReq *req = [[SendAuthReq alloc ] init];  
    req.scope = @"snsapi_userinfo,snsapi_base";  
    req.state = @"0744" ;  
    [WXApi sendReq:req];  
}
```

这里获取后会调用之前在AppDelegate里面的对应`oauthResp`回调，获得得到的`code`。

第二步：token和openid

```
-(void)getAccess_token  
{  
    //https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code  
    NSString *url = [NSString stringWithFormat:@"https://api.weixin.qq.com/sns/oauth2/access_token?appid=%@&secret=%@&code=%@&grant_type=authorization_code",kWXAPP_ID,kWXAPP_SECRET,self.wxCode.text];   dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{  
        NSURL *zoneUrl = [NSURL URLWithString:url];  
        NSString *zoneStr = [NSString stringWithContentsOfURL:zoneUrl encoding:NSUTF8StringEncoding error:nil];  
        NSData *data = [zoneStr dataUsingEncoding:NSUTF8StringEncoding];  
        dispatch_async(dispatch_get_main_queue(), ^{  
            if (data) {  
                NSDictionary *dic = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingMutableContainers error:nil];  
              /* 
               { 
               "access_token" = "OezXcEiiBSKSxW0eoylIeJDUKD6z6dmr42JANLPjNN7Kaf3e4GZ2OncrCfiKnGWiusJMZwzQU8kXcnT1hNs_ykAFDfDEuNp6waj-bDdepEzooL_k1vb7EQzhP8plTbD0AgR8zCRi1It3eNS7yRyd5A"; 
               "expires_in" = 7200; 
               openid = oyAaTjsDx7pl4Q42O3sDzDtA7gZs; 
               "refresh_token" = "OezXcEiiBSKSxW0eoylIeJDUKD6z6dmr42JANLPjNN7Kaf3e4GZ2OncrCfiKnGWi2ZzH_XfVVxZbmha9oSFnKAhFsS0iyARkXCa7zPu4MqVRdwyb8J16V8cWw7oNIff0l-5F-4-GJwD8MopmjHXKiA"; 
               scope = "snsapi_userinfo,snsapi_base"; 
               } 
               */   
                self.access_token.text = [dic objectForKey:@"access_token"];  
                self.openid.text = [dic objectForKey:@"openid"];   
            }  
        });  
    });  
}
```

利用GCD来获取对应的`token`和`openID`.

第三步：userinfo

```
-(void)getUserInfo  
{  
   // https://api.weixin.qq.com/sns/userinfo?access_token=ACCESS_TOKEN&openid=OPENID  
    NSString *url = [NSString stringWithFormat:@"https://api.weixin.qq.com/sns/userinfo?access_token=%@&openid=%@",self.access_token.text,self.openid.text];  
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{  
        NSURL *zoneUrl = [NSURL URLWithString:url];  
        NSString *zoneStr = [NSString stringWithContentsOfURL:zoneUrl encoding:NSUTF8StringEncoding error:nil];  
        NSData *data = [zoneStr dataUsingEncoding:NSUTF8StringEncoding];  
        dispatch_async(dispatch_get_main_queue(), ^{  
            if (data) {  
                NSDictionary *dic = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingMutableContainers error:nil];  
                /*{ 
                 city = Haidian; 
                 country = CN; 
                 headimgurl = "http:/wx.qlogo.cn/mmopen/FrdAUicrPIibcpGzxuD0kjfnvc2klwzQ62a1brlWq1sjNfWREia6W8Cf8kNCbErowsSUcGSIltXTqrhQgPEibYakpl5EokGMibMPU/0"; 
                 language = "zh_CN"; 
                 nickname = "xxx"; 
                 openid = oyAaTjsDx7pl4xxxxxxx; 
                 privilege =     ( 
                 ); 
                 province = Beijing; 
                 sex = 1; 
                 unionid = oyAaTjsxxxxxxQ42O3xxxxxxs; 
                 }*/   
                self.nickname.text = [dic objectForKey:@"nickname"];  
                self.wxHeadImg.image = [UIImage imageWithData:[NSData dataWithContentsOfURL:[NSURL URLWithString:[dic objectForKey:@"headimgurl"]]]];  
            }  
        });  
    });  
}
```                

执行到这一步就算完成了整个授权登录的功能，能把昵称和头像显示出来，剩下的就是及时刷新你的token，详情可参考开发文档。

评价：微信的开发文档相比容易理解和调试，虽然没有demo，但是文档比较详细，所以可以在一定程度上减轻了开发的困难，但是相比之下微信的授权步骤比较麻 烦，需要三步才能彻底获取用户信息，这点没有QQ和新浪微博简洁，需要有一定的阅读和代码功底，希望能给大家带来帮助。
