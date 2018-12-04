---
layout: post                        ## 文章使用的模板
title: iOS中的UIApplication和视图		## 文章的标题
date: 2018-11-30					## 发布时间
author: "FaceWaller"                ## 作者
categories: iOS
tags: iOS
---

iOS视图相关的内容 !!!!!

# UIApplication

## 简介
>UIApplication对象是应用程序的象征,一个UIApplication对象就代表一个应用程序.每一个应用的UIApplication都是单例的,如果试图在程序中新建一个UIApplication对象,那么将报错.  
>可以通过[UIApplication sharedApplication]获取这个单例对象,可以利用UIApplication对象,进行一些应用级别的操作.

## 应用级别的操作
>设置应用图标的红色提醒数字

	UIApplication * app = [UIApplication sharedApplication];
	app.applicationIconBadgeNumber = 1;

>设置联网指示器的可见性

	app.networkActivityIndicatorVisible=YES;
	
>管理状态栏

	//设置状态栏样式
	app.statusBarStyle = UIStatusBarStyleDefault;
	//设置状态栏是否隐藏
	app.statusBarHidden = YES;
	
	//也可以通过UIViewController进行单独的状态栏管理.  
	状态栏的样式　　 - (UIStatusBarStyle)preferredStatusBarStyle; 
	状态栏的可见性　 - (BOOL)prefersStatusBarHidden;  

>openURL 方法

	//打电话
	[app openURL: [NSURL URLWithString:@"tel://10086"]];
	//发短信
	[app openURL: [NSURL URLWithString:@"sms://10086"]];
	//发邮件
	[app openURL: [NSURL URLWithString:@"mailto://abc@163.com"]];
	//打开网页 
	[app openURL: [NSURL URLWithString:@"http://www.baidu.com"]];
	//打开其他app
	[app openURL: [NSURL URLWithString:@"weixin://"]];

## UIApplication Delegate

> 当应用程序将要入非活动状态执行，在此期间，应用程序不接收消息或事件，比如来电话了  

	- (void)applicationWillResignActive:(UIApplication *)application
	
> 当应用程序入活动状态执行，这个刚好跟上面那个方法相反

	-  (void)applicationDidBecomeActive:(UIApplication *)application
	
> 当程序被推送到后台的时候调用。所以要设置后台继续运行，则在这个函数里面设置即可
	
	- (void)applicationDidEnterBackground:(UIApplication  *)application
	
> 当程序从后台将要重新回到前台时候调用，这个刚好跟上面的那个方法相反。

	-  (void)applicationWillEnterForeground:(UIApplication *)application
	
> 当程序将要退出是被调用，通常是用来保存数据和一些退出前的清理工作。这个需要要设置UIApplicationExitsOnSuspend的键值。

	-  (void)applicationWillTerminate:(UIApplication *)application
	
> iPhone设备只有有限的内存，如果为应用程序分配了太多内存操作系统会终止应用程序的运行，在终止前会执行这个方法，通常可以在这里进行内存清理工作防止程序被终止

	-  (void)applicationDidReceiveMemoryWarning:(UIApplication *)application
	
> 当系统时间发生改变时执行

	- (void)applicationSignificantTimeChange:(UIApplication*)application
	
> 当程序载入后执行

	- (void)applicationDidFinishLaunching:(UIApplication*)application

> 当StatusBar框将要变化时执行
	
	- (void)application:(UIApplication)application  willChangeStatusBarFrame:(CGRect)newStatusBarFrame
	
> 当StatusBar框变化完成后执行

	-  (void)application:(UIApplication*)application didChangeSetStatusBarFrame:(CGRect)oldStatusBarFrame

> 当StatusBar框方向将要变化时执行
	
	- (void)application:(UIApplication *)application willChangeStatusBarOrientation:(UIInterfaceOrientation)newStatusBarOrientation duration:(NSTimeInterval)duration
	
> 当StatusBar框方向变化完成后执行

	-  (void)application:(UIApplication*)application  didChangeStatusBarOrientation:(UIInterfaceOrientation)oldStatusBarOrientation
	
> 当通过url执行
	
	-  (BOOL)application:(UIApplication*)application handleOpenURL:(NSURL*)url


# UIWindow

## 简介
>UIWindow定义了一个负责管理协调一个App的View在硬件屏幕显示的窗口类,是一个特殊的UIView类.
>作用: 1.提供一块给View的显示区域.  2.分发事件给View.

## 新建一个window:
	
>默认情况下,一个App只有一个UIWindow.但即使我们什么都不做也会有其他的UIWindow:

	1.键盘对应的UITextEffectWindow.
	2.状态栏对应的UIStatusBarWindow

>需要创建多UIWindow的情况:
	
	1.全局性的自定义HUD,alert效果.
	2.需要展示的界面盖住UIstatusBar.
	
>创建一个window

	//创建一个window对象,并强持有它
	UIWindow * win = [[UIWindow alloc]initWithFrame:CGRectMake(0, 0, 100, 100)];
	self.win = win;
	win.windowLevel = UIWindowLevelAlert;
			
	//创建一个控制器赋值为window的根控制器
	UIViewController * vc = [[UIViewController alloc]init];
	win.rootViewController = vc;
	
	//显示窗口
	[win makeKeyAndVisible];
	
>销毁一个window
	
	[win resignKeyWindow];
	self.win = nil;

# 响应者链条
>App使用响应者对象接收和处理事件,响应者对象是任何UIResponder的实例.UIResponder的子类包括UIView UIViewController UIApplication等.响应者接收到原始事件数据,处理事件或者转发给另一个响应者对象.当app接收到一个时间时,UIKit自动引导事件到最合适的响应者对象即第一响应者.


![avatar](https://upload-images.jianshu.io/upload_images/2492441-d33cd8efed539a33.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

>相应者的传递都是从父类往子类去找,其中有两个关键方法.

	hitTest:withEvent:          //寻找响应者
	pointInside:withEvent:      //判断点击处是否在视图内

>通过重写这两个方法,可以对事件的传递施加些影响.如不规则图形的点击,扩大缩小点击范围.

# UIView

## UIView简介
>UIView是窗口上的一块区域.我们在app中所有能看见的都是UIView或者它的子类实例.

## UIView的作用

- 负责内部区域的内容渲染
- 负责内部区域的触摸事件
- 管理本身的所有子视图
- 处理基本的动画


## CALayer
>UIView之所以能够显示在屏幕上,完全是因为它内部的一个图层,在创建UIView对象时,UIView内部会自动创建一个图层即CALayer对象,通过UIView的layer属性可以访问这个层.

>当UIView需要显示到屏幕上时,会调用drawRect:方法进行绘图,然后将内容绘制在自己的图层上,然后系统会将图层拷贝到屏幕上显示.也就是说UIView是通过CALayer实现显示功能的.

>通过直接操作CALayer,可以方便的调整UIView的一些外观,比如阴影,圆角,边框等.还可以通过CALayer的contents属性来设置图片.

>通过CALayer能够实现的动画也比UIView动画更加丰富.



