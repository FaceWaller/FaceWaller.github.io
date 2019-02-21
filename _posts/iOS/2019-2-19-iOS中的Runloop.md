---
layout: post                        ## 文章使用的模板
title: iOS中的Runloop  				## 文章的标题
date: 2019-2-19						## 发布时间
author: "FaceWaller"                ## 作者
categories: iOS
tags: iOS Runloop
---

# 定义

	
Apple官方给runloop定义: runloop是线程的基础支撑,是循环处理事件的机制,一个具体的runloop就是一个事件处理循环.

runloop的目的是使线程在没事情可做时进入休眠状态,避免CPU空转.


Runloop Modes

runloop mode是事件源的集合 + runloop观察者的集合   runloop每次都运行在某个特定的mode上





http://zxfcumtcs.github.io/2014/11/15/runloop/