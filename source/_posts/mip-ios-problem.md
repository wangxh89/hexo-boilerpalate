---
title: 记 开发MIP IOS版本中遇到的坑
date: 2015-3-15 16:07:57
tags: ios
categories: ios
---
## storyboard中使用 TTTattibutedLabel 及didSelectLinkWithURL没有调用问题
### storyboard中使用 TTTattibutedLabel
Go to Storyboard and select the UILabel and change its class to TTTAttributedLabe
关联代码
```
  @property (weak, nonatomic) IBOutlet TTTAttributedLabel *tips;
```
增加链接
```cpp
   self.tips.enabledTextCheckingTypes = NSTextCheckingTypeLink;
  self.tips.delegate = self;
  self.tips.font = [UIFont systemFontOfSize:12];

  [self.tips setText:_(@"注册即视为同意《服务协议》")]; // 注册即视为同意《服务协议》
  NSRange range = [self.tips.text rangeOfString:@"服务协议"];
  [self.tips addLinkToURL:[NSURL URLWithString:@"http://10.45.4.30:8080/mip/m/userLincese"] withRange:range];
处理回调
  #pragma mark - TTTAttributedLabelDelegate

  - (void)attributedLabel:(TTTAttributedLabel *)label didSelectLinkWithURL:(NSURL *)url
  {
      [[[UIActionSheet alloc] initWithTitle:[url absoluteString] delegate:self cancelButtonTitle:_(@"CANCEL") destructiveButtonTitle:nil otherButtonTitles:_(@"OPEN_LINK"), nil] showInView:self.view];
  }

  #pragma mark - UIActionSheetDelegate

  - (void)actionSheet:(UIActionSheet *)actionSheet clickedButtonAtIndex:(NSInteger)buttonIndex {
      if (buttonIndex == actionSheet.cancelButtonIndex) {
          return;
      }

      [[UIApplication sharedApplication] openURL:[NSURL URLWithString:actionSheet.title]];
  }
```

### didSelectLinkWithURL没有调用问题
#### 第一次处理
参考连接http://stackoverflow.com/questions/17796512/ios-tttattributedlabel-delegate-didselectlinkwithurl-not-getting-called
把 “User Interaction Enabled” 给打钩 ，并在代码中设置
`label.userInteractionEnabled = YES;`
没有效果

### 第二次处理
发现我在代码中侦听了整个视图的Tap事件
```cpp
(void)addTap
{
UITapGestureRecognizer * tapGestuer =
[[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(hiddenKeyboard:)];
tapGestuer.numberOfTapsRequired = 1;
tapGestuer.delegate = self;
[self.view addGestureRecognizer:tapGestuer];
}
-(void)hiddenKeyboard:(UITapGestureRecognizer*)gesture
{
[self.view endEditing:YES];
}
```
UITapGestureRecognizer会隐藏了一切能响应用户Tap的控件
应当是这个问题，导致TTTattibutedLabel 侦听不到事件，所以无法在回调中使用.
还好UITapGestureRecognizer有过滤功能，它能够让你选择哪些控件需要响应UITapGestureRecognizer哪些不需要
处理措施如下：
实现UIGestureRecognizerDelegate委托的如下方法，YES是需要响应，NO为不需要响应
```
//如果输入框都没有获取焦点，也就是没有弹出键盘，则都不需要响应

#pragma mark -
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch
{
    if (![self.userNameTextField isFirstResponder] && ![self.userPassTextField isFirstResponder] && ![self.verifyCodeTextField isFirstResponder]) {
        return NO;
    }
    return YES;
}
```

## 使用AFNetworking加载图片问题
### 问题
```
NSURL *URL = [NSURL URLWithString:@"http://10.45.4.30:8080/mip/m/getImageFile?id=xxxxxxx"];
[imageView setImageWithURL:URL];
```

上面的代码显示不出来图片的，查了很久，试了各种方法

### 原因
http://10.45.4.30:8080/mip/m/getImageFile?id=xxxxxxx这种形式后台返回的图片的二进制流。而setImageWithURL 需要[request addValue:@”image/“ forHTTPHeaderField:@”Accept”]; 也就是说只接收image/的头，，而在JAVA后台代码中只返回的图片的二进制流

### 处理
使用 – setImageWithURLRequest:placeholderImage:success:failure:
```cpp
NSURL * imageUrl = [NSURL URLWithString:[NSString stringWithFormat:@"%@?id=%@",kUrlGetImageFile,introPic]];
NSURLRequest * request = [[NSURLRequest alloc] initWithURL:imageUrl];

[cell.image setImageWithURLRequest:request placeholderImage:_defaultImage success:^(NSURLRequest *request, NSHTTPURLResponse *response, UIImage *image) {
    cell.image.image = image;
} failure:^(NSURLRequest *request, NSHTTPURLResponse *response, NSError *error) {
    LogOut("%@显示图片出错,错误信息:%@",imageUrl,error);
}];
```
### 参考
http://zhangbuhuai.com/2014/08/03/AFNetworking-20-Tutorial/
http://blog.shiqichan.com/using-afnetworking-sdwebimage-and-ohhttpstubs
