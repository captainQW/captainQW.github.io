title: 使用safari对webview进行调试
date: 2016-12-06 13:01:47
categories: linux
tags: [ios]
---

在web开发的过程中，抓包、调试页面样式、查看请求头是很常用的技巧。其实在iOS开发中，这些技巧也能用（无论是模拟器还是真机），不过我们需要用到mac自带的浏览器Safari。所以，本文将讲解如何使用Safari对iOS程序中的webview进行调试。


环境信息：
Mac OS X 10.10.1
Xcode 6.1.1
iOS 8.1


1、打开模拟器（真机）的开发者模式

【设置】->【Safari】->【高级】->【Web检查器】打开

![打开iphone设备中的web检查器](https://static.verycloud.cn/sites/default/files/images/ios-user-safari-debug-webview-4.png)
打开iphone设备中的web检查器

2、打开Mac上Safari的开发者模式

【Safari】->【偏好设置】->【高级】->【在菜单栏中显示“开发”菜单】勾选

![打开Safari中的开发者模式](https://static.verycloud.cn/sites/default/files/images/ios-user-safari-debug-webview-2.png)
打开Safari中的开发者模式

3、写一个webview并加载一个网页

```bash
#import "ViewController.h"
@interfaceViewController ()

@property (strong, nonatomic) UIWebView *webView;

@end

@implementation ViewController

 - (void)viewDidLoad {
    
    [superviewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    
    _webView = [[UIWebViewalloc] initWithFrame:self.view.bounds];
    [_webView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.baidu.com"]]];
    [self.view addSubview:_webView];
 }

@end
```

4、在模拟器（真机）中打开webview应用，并打开Safari查看网络信息

【开发】->【iOS Simulator】->【正在调试的网站】

注意：必须要webview在加载网页时，打开Safari才可以看到调试模式。


![打开Safari中的调试](https://static.verycloud.cn/sites/default/files/images/ios-user-safari-debug-webview-6.png)
打开Safari中的调试
在弹出的调试窗口中，可以看到当前正在加载网页的各种信息，包括源码、请求头、图片、加载的资源与脚本、控制台输出等。并且它和web前端的调试方式相同，你可以直接修改网页的CSS样式，对网页布局等进行修改，而不用重新运行整个App。

5、修改web样式

将光标选中到要修改的样式，进行修改后，可以直接在模拟器中看到修改后的效果。

![直接修改webview中的样式](https://static.verycloud.cn/sites/default/files/images/ios-user-safari-debug-webview-3.png)

这样就可以直接修改webview中的样式。
