---
layout: post                        ## 文章使用的模板
title: iOS中的runtime详解  			## 文章的标题
date: 2018-11-20					## 发布时间
author: "FaceWaller"                ## 作者
categories: iOS
tags: iOS runtime
---

偏OC底层的知识 !!!!!

# runtime基本信息
>作为动态语言,runtime是oc区别于其他静态语言的重要特征.在静态语言中,函数的调用会在编译期就已经决定好,在编译完成后直接顺序执行.但是OC是一门动态语言,函数的调用变成了消息发送,在编译期间不能知道要调用哪个函数.runtime就是解决如何在运行时期找到调用方法的问题.

# 从runtime的角度看类与对象

>在OC中,每一个对象都是某个类的实例,且这个对象的isa指针指向它所属的类.  
>在objc.h中可以看到实例的结构,实质上就是一个结构体.

	/// Represents an instance of a class.
	struct objc_object {
		Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
	};
	
>类的isa指针指向它的元类.可以说本质上类也是对象,它是元类的实例.    
>在runtime.h中可以看到类的结构,实质上也是一个结构体.

	struct objc_class {
		Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

	#if !__OBJC2__
		Class _Nullable super_class; //父类
		const char * _Nonnull name ; //类名
		long version; //类的版本信息
		long info; //类信息
		long instance_size; //类的实例变量大小
		struct objc_ivar_list * _Nullable ivars; //类的成员变量链表
		struct objc_method_list * _Nullable * _Nullable methodLists; //方法链表
		struct objc_cache * _Nonnull cache; //方法缓存
		struct objc_protocol_list * _Nullable protocols; //协议链表
	#endif

	} OBJC2_UNAVAILABLE;
	
>正如类是一个对象一样,元类也是一个对象,它的isa指向根元类.

>实例,类,元类的关系可以用一张图来阐述

![avatar](http://blog.leichunfeng.com/images/object_model.png)

# 消息机制

## SEL与IMP
>SEL:类成员方法的指针,但不同于C语言中的函数指针,函数指针直接保存了方法的地址,但SEL只是方法编号.  
>IMP:一个函数指针,保存了方法的地址.

	SEL selector = @selector(test); //获取方法编号
	[self performSelector:selector]; //根据SEL执行方法
	NSString * selStr = NSStringFromSelector(selector); //获取方法名
	IMP imp = [self methodForSelector:selector]; //IMP 就是函数指针
	imp(); //执行方法

## 方法寻找
>在OC中,对象调用方法,实际就是向对象发送消息,通过runtime来进行具体方法的调用.底层的具体实现,是用C语言函数实现的.

	//第一个参数代表接收者,第二个代表函数名,后面是参数
	void objc_msgSend(id self, SEL cmd, ...)
	
>objc_msgSend函数会在接受者所属的类中搜寻列表,找到方法后就跳转到其实现代码.如果没找到,就沿着Superclass的路线找下去.这也就是子类方法优先父类方法执行的原因.

## 转发机制
>正常情况下,在找不到方法时,app就会crash.但是在崩溃之前,系统还提供了转发机制来避免崩溃.

![avatar](https://upload-images.jianshu.io/upload_images/588630-9fb706d219d3b412.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

### 动态方法解析
>判断本类是否能够处理sel方法,能处理就返回yes  
>可以在动态添加方法的同时重写该方法,同时返回YES

	+ (BOOL)resolveInstanceMethod:(SEL)sel  //实例方法
	+ (BOOL)resolveClassMethod:(SEL)sel   //类方法
	
	
	void myMethod(id self,SEL _cmd){
		NSLog(@"This is added dynamic");
	}
	+ (BOOL)resolveInstanceMethod:(SEL)sel{
		if (sel == @selector(sendMessage)) {
			/** 动态添加方法
			 *  class_addMethod  对应的参数如下：
			 *  cls 想要添加的类
			 *  name 添加后的方法selector名字
			 *  imp 具体的方法实现
			 *  types 方法参数的编码 详见：https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1
			 *
			 */
			class_addMethod([self class], sel, (IMP)myMethod, "v@:");
			return YES;
		}else{
			return [super resolveInstanceMethod:sel];
		}
	}

### 快速转发
>快速转发可以替换消息的接受者为其他对象
	
	- (id)forwardingTargetForSelector:(SEL)aSelector{
	#pragma clang diagnostic push
	#pragma clang diagnostic ignored "-Wundeclared-selector"
		if (aSelector == @selector(sendMessage)) {
			return self.tar;
		}else{
			return [super forwardingTargetForSelector:aSelector];
		}
	}
	//self.tar 是我设置的靶对象,所有的崩溃都指向它,可以用来收集崩溃信息


### 完整转发
>相比快速转发,完整转发可以做更多的事情 
	
	
	- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
		NSMethodSignature *methodSignature = [super methodSignatureForSelector:aSelector];
		if(!methodSignature){
			return [NSMethodSignature signatureWithObjCTypes:"v@:"];
		}
		return methodSignature ;
	}
	
	-(void)forwardInvocation:(NSInvocation *)anInvocation{
		if ([self.tar respondsToSelector:[anInvocation selector]]) {
			[anInvocation invokeWithTarget:self.tar];
		}else{
			[super forwardInvocation:anInvocation];
		}
	}
	
	//forwardInvocation:方法就是一个不能识别消息的分发中心,将这些不能识别的消息转发给不同的消息对象,或者转发给同一个对象,或者什么都不做也可以.
	
>快速转发和完整转发的对比

- 需要重载的api方法不同  
	快速转发只需要重载一个API,完整转发需要重载两个API.  
	前者只需在API方法里面返回一个新对象即可,后者需要对被转发的消息进行重签并手动转发给新对象.
-  转发给新对象的个数不同
	前者只能转发一个对象,后者可以转发给多个对象.

# C语言的api

## 获取一个属性名称
	objc_property_t property_t = class_getProperty([StudyMessageSonObj class], "name");
	const char * name = property_getName(property_t);
	NSLog(@"%s",name);
	
## 拷贝属性列表
	unsigned int outCount;
	objc_property_t * properties = class_copyPropertyList([StudyMessageObj class], &outCount);
	for (int i = 0; i<outCount; i++) {
		objc_property_t subProperty_t = properties[i];
		const char * subName = property_getName(subProperty_t);
		NSLog(@"%s",subName);
	}

## 动态添加属性
	BOOL class_addProperty(Class cls, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount)


## 动态添加"属性"的另一种方式(进行对象的联合)
>利用 objc_setAssociatedObject 生成的关联对象无法直接利用目前主流的Json转Model库(原因是无法在ivar及property中遍历出来)。

	static void *key = "key";
	- (void)setName:(NSString *)name{
		objc_setAssociatedObject(self, key, name, OBJC_ASSOCIATION_COPY_NONATOMIC);
	}
	- (NSString *)name{
		return objc_getAssociatedObject(self, key);
	}
	
	
## 动态替换属性
	void class_replaceProperty(Class cls, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount)

## 获取属性的一些信息
	const char *property_getName(objc_property_t property)
	const char *property_getAttributes(objc_property_t property)
	
## 获得一个实例方法 类方法

	Method class_getInstanceMethod(Class cls, SEL name)
	Method class_getClassMethod(Class cls, SEL name)

## 方法实现相关操作

	IMP class_getMethodImplementation(Class cls, SEL name)
	IMP method_setImplementation(Method m, IMP imp)
	void method_exchangeImplementations(Method m1, Method m2)

## 拷贝方法列表

	Method *class_copyMethodList(Class cls, unsigned int *outCount)


## 动态添加方法

	BOOL class_addMethod(Class cls, SEL name, IMP imp, const char *types)

## 动态替换方法

	IMP class_replaceMethod(Class cls, SEL name, IMP imp, const char *types)


## 获取方法的相关信息

	SEL method_getName(Method m)
	IMP method_getImplementation(Method m)
	const char *method_getTypeEncoding(Method m)
	unsigned int method_getNumberOfArguments(Method m)
	char *method_copyReturnType(Method m)
	char *method_copyArgumentType(Method m, unsigned int index)


## 选择器相关

	const char *sel_getName(SEL sel)
	SEL sel_registerName(const char *str)

## 用block作为方法实现

	IMP imp_implementationWithBlock(id block)
	id imp_getBlock(IMP anImp)
	BOOL imp_removeBlock(IMP anImp)
	
	
