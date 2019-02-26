---
layout: post                        ## 文章使用的模板
title: iOS中的Runloop  				## 文章的标题
date: 2018-8-19						## 发布时间
author: "FaceWaller"                ## 作者
categories: iOS
tags: iOS Runloop
---

# 基本信息

## 定义
>Apple官方给runloop定义: runloop是线程的基础支撑,是循环处理事件的机制,一个具体的runloop就是一个事件处理循环.

## 目的
>runloop的目的是使线程在没事情可做时进入休眠状态,避免CPU空转.

# Runloop Modes
>runloop mode是事件源的集合 + runloop观察者的集合   runloop每次都运行在某个特定的mode上

>runloopmode可以在runloop监听过程中过滤掉不关心的事件源，专注于某些特定的事件。

<!--![avatar](http://blog.leichunfeng.com/images/object_model.png)-->

当有 UI 滑动事件时，系统会将main runloop切换到NSEventTrackingRunLoopMode，以限定此时的事件源，确保滑动的流畅性.


NSRunloopCommoModes 包含NSEventTrackingRunLoopMode。
默认情况下，子线程想通过performSelectoreOnMainThread或dispatch在主线程执行selector，会关联到NSRunloopCommoModes，如果此时有UI滑动，会影响流畅性。




# Runloop 与 thread
每个thread都有自己的runloop，可以通过NSRunLoop的类方法currentRunLoop获取当前线程的runloop，但只有主线程的runloop是默认开启的，其他线程如果希望存活，需要手动开启runloop。

什么情况需要开启线程的runloop？ 基本原则：线程需要处理异步事件。
大概几种情况：
* 需要通过端口或自定义输入源或者与其他线程通讯
* 在线程中需要使用定时器
* 在线程上使用performSelector系列方法
* 需要线程周期性的执行一些任务


在开启runloop前，至少要绑定一个事件源到runloop上，否者runloop将直接退出
事件源大概有四类：
* Port-Based Sources
* Custom Input Sources
* Cocoa Perform Selector Sources
* Timer Sources
也可以归纳为两类  内部事件（timer）和外部事件。

注意点:
* 在子线程执行performSelector时，子线程需要开启runloop，不然selector将会入队，直到runloop启动才会被执行
* 所有入队的performSelector将会在一次runloop中全部被处理，而不是每次runloop处理一个selector
* 当selector执行后，该performSource将会被从runloop上移除掉
* 当timer触发的事件不会使runloop退出
* 当timer触发时，runloop正在处理其他事件，timerhandler需要等到下一个loop才会被执行
* 如果runloop没有启动，timer永远不会触发


# 生命周期
runloop的生命周期大概要经历三个阶段：
* runloop启动后首先处理待处理的事件，如：timer    inputsource
* 进入休眠，等待新任务的到来
* 有新任务了，runloop被唤醒处理新任务，runloop重启或退出


# runloopObserVer

通过runloop observer，我们可以近距离的监听runloop以下重要事件：

* runloop 启动
* runloop 即将处理timer
* runloop 即将处理 input source event
* runloop 即将进入休眠状态
* runloop 被唤醒，但还未处理唤醒它的事件
* runloop 退出

CFRunLoopObserver 也被称为 runloop之窗

通过添加检测方法可以查看runloop的各阶段状态，定义的observerRunLoop()可以随意添加到主线程或者子线程。

	void observerRunLoop()
	{
		CFRunLoopObserverRef observer = CFRunLoopObserverCreateWithHandler(kCFAllocatorDefault, kCFRunLoopAllActivities, YES, 0, ^(CFRunLoopObserverRef observer, CFRunLoopActivity activity) {

		switch (activity) {
			case kCFRunLoopEntry:
				NSLog(@"进入");
				break;
			case kCFRunLoopBeforeTimers:
				NSLog(@"即将进入timer事件");
				break;
			case kCFRunLoopBeforeSources:
				NSLog(@"即将进入source事件");
				break;
			case kCFRunLoopBeforeWaiting:
				NSLog(@"休眠");
				break;
			case kCFRunLoopAfterWaiting:
				NSLog(@"被唤醒");
				break;
			case kCFRunLoopExit:
				NSLog(@"退出");
				break;

			default:
				break;
			}

		});
		CFRunLoopAddObserver([[NSRunLoop currentRunLoop] getCFRunLoop], observer, kCFRunLoopCommonModes);
	}
