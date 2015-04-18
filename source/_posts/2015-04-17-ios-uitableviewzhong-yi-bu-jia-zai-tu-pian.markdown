---
layout: post
title: "IOS UITableView中异步加载图片"
date: 2015-03-25 10:46:11 +0800
comments: true
categories: ios扩展
---
在平时做项目时，经常会遇到关于UITableView加载网络图片的问题，一下是几种常用的解决方案：

1.使用第三方类库：[SDWebImage](https://github.com/rs/SDWebImage)

首先下载该类库，然后添加到自己的项目工程中，具体使用代码如下：

<!--more-->

```ruby
 #import <SDWebImage/UIImageView+WebCache.h>
 ...
 
 - (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
  static NSString *MyIdentifier = @"MyIdentifier";

  UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:MyIdentifier];

  if (cell == nil)
  {
    cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault
                     reuseIdentifier:MyIdentifier];
  }

  // Here we use the new provided setImageWithURL: method to load the web image
  [cell.imageView setImageWithURL:[NSURL URLWithString:@"http://www.domain.com/path/to/image.jpg"]
           placeholderImage:[UIImage imageNamed:@"placeholder.png"]];

  cell.textLabel.text = @"My Text";
  return cell;
}

```

2.使用iOS自有的异步多线程，下载好图片后存入cache，再使用delegate函数更新tableview：

- (void)reloadRowsAtIndexPaths:(NSArray *)indexPaths withRowAnimation:(UITableViewRowAnimation)animation

初始化：

```
@interface ViewController ()

@property (strong, nonatomic)  NSMutableArray *imageArray;
@property (strong, nonatomic) NSCache *imageCache;
@property (weak, nonatomic) IBOutlet UITableView *myCustomTableView;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.imageCache = [[NSCache alloc] init];
}
```

TableView delegate函数：

```
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
  return 1;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
  // Number of rows is the number of time zones in the region for the specified section.
  return 32;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
  NSLog(@"cellForRowAtIndexPath %d", indexPath.row);
  
  static NSString *MyIdentifier = @"customIdentifier";
  UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:MyIdentifier];
  if (cell == nil) {
    cell = [[CustomTableViewCell alloc] initWithStyle:UITableViewCellStyleDefault  reuseIdentifier:MyIdentifier];
  }
  
  if ([cell isKindOfClass:[CustomTableViewCell class]])
  {
    CustomTableViewCell *customCell = (CustomTableViewCell *)cell;
    NSData *imageData = [self.imageCache objectForKey:[NSString stringWithFormat:@"%d", indexPath.row]];
    
    if(imageData != nil){
      customCell.customImageView.image = [UIImage imageWithData:imageData];
    }
    else{
      customCell.customImageView.image = [UIImage imageNamed:@"test1"];
      
      dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, (unsigned long) NULL), ^(void){
        UIImage *image = [self getPictureFromServer];
        
        if (image != nil) {
          NSData *tmpImageData = UIImagePNGRepresentation(image);
          [self.imageCache setObject:tmpImageData forKey:[NSString stringWithFormat:@"%d", indexPath.row]];
          NSLog(@"download image for %d", indexPath.row);
          
          [self performSelectorOnMainThread:@selector(reloadTableViewDataAtIndexPath:) withObject:indexPath waitUntilDone:NO];
        }
        else{
          // do nothing ..
        }
      });
    }
  }
  else{
    // do nothing ..
  }

  return cell;
}
```

reloadTableViewDataAtIndexPath：

```
- (void) reloadTableViewDataAtIndexPath:(NSIndexPath *)indexPath{
  NSLog(@"MyWineViewController: in reload collection view data for index: %d", indexPath.row);

  NSArray *indexArray = [NSArray arrayWithObjects:indexPath, nil];
  [self.myCustomTableView reloadRowsAtIndexPaths:indexArray withRowAnimation:UITableViewRowAnimationFade];
//	[self.myCustomTableView reloadRowsAtIndexPaths:indexArray withRowAnimation:UITableViewRowAnimationAutomatic];
//	[self.myCustomTableView reloadRowsAtIndexPaths:indexArray withRowAnimation:UITableViewRowAnimationBottom];
//	[self.myCustomTableView reloadRowsAtIndexPaths:indexArray withRowAnimation:UITableViewRowAnimationLeft];
//	[self.myCustomTableView reloadRowsAtIndexPaths:indexArray withRowAnimation:UITableViewRowAnimationMiddle];
//	[self.myCustomTableView reloadRowsAtIndexPaths:indexArray withRowAnimation:UITableViewRowAnimationNone];
//	[self.myCustomTableView reloadRowsAtIndexPaths:indexArray withRowAnimation:UITableViewRowAnimationRight];
//	[self.myCustomTableView reloadRowsAtIndexPaths:indexArray withRowAnimation:UITableViewRowAnimationTop];
}
```

下载图片

```
- (UIImage *) getPictureFromServer{
  UIImage *image = nil;
  NSString *urlString = [NSString stringWithFormat:@"http://yourPicture<span style="font-family: Arial, Helvetica, sans-serif;">.</span><span style="font-family: Arial, Helvetica, sans-serif;">jpg"];</span>
  
  NSMutableURLRequest *request = [[NSMutableURLRequest alloc] init];
  [request setURL:[NSURL URLWithString:urlString]];
  [request setHTTPMethod:@"GET"];
  
  NSURLResponse *response;
  NSError *connectionError;
  NSData *returnData = [NSURLConnection sendSynchronousRequest:request returningResponse:&response error:&connectionError];

  if (connectionError == NULL) {
    NSInteger statusCode = [(NSHTTPURLResponse *)response statusCode];
    NSLog(@"getPictureFromServer - Response Status code: %ld", (long)statusCode);
    
    if (statusCode == 200) {
      image = [UIImage imageWithData:returnData];
    }
    else{
      // do nothing..
    }
  }
  else{
    // do nothing..
  }
  
  return image;
}
```

小结：通过上述两种方法都可以解决UITableView异步加载图片的问题，我也是因为图片异步加载的问题纠结了很久，所以就总结了这篇博文，希望能够帮助到其他人。

原文：[查看原文](http://blog.csdn.net/willyang519/article/details/41833293)

