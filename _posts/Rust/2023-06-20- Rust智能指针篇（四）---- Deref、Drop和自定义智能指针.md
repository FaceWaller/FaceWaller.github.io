---
layout: post                        ## 文章使用的模板
title: Rust智能指针篇（四）---- Deref、Drop和自定义智能指针				## 文章的标题
date: 2023-06-20						## 发布时间
author: "FaceWaller"                ## 作者
categories: Rust
tags: Rust
---

Deref和Drop与智能指针之间存在密切的关系，但它们并不是智能指针本身。它们是智能指针的一部分，用于提供额外的功能和控制。

Deref

Deref可以让智能指针像引用那样工作

Deref是一个trait，允许你重载解引用运算符*，以便通过智能指针访问其内部的数据。智能指针类型可以通过实现Deref trait 来提供对内部数据的解引用操作。这使得智能指针的行为类似于指针，允许你使用指针操作符来访问其内部的数据。

Deref的位置位于std::ops::Deref; 其完整定义如下：

pub trait Deref {
    type Target: ?Sized;
    fn deref(&self) -> &Self::Target;
}
可以看到该trait只有一个函数deref，首先定义返回类型Target，然后实现deref函数就ok了；后面我们通过自定义智能指针来进一步了解；

Drop

Drop的作用是用来释执行清理操作的

Drop是一个trait，允许你定义类型在离开作用域时需要执行的清理操作。智能指针通常需要在离开作用域时执行特定的清理逻辑，例如释放内存或关闭资源。通过实现Drop trait，智能指针可以定义其自己的析构函数，以确保在合适的时机执行清理操作。

在rust中，内存的自动释放是通过所有权和生命周期规则来实现的，一般情况下不需要我们去手动管理；然而有些情况下需要更细颗粒度的资源管理和清理操作，这就需要使用Drop trait,比如释放特定资源、关闭文件、发送网络请求等；这些清理操作可能不仅仅涉及内存的释放，还包括其他与资源相关的任务；

Drop的位置位于std::ops::Drop；其完整定义如下

pub trait Drop {
    fn drop(&mut self);
}
同样，该trait也只有一个函数drop；在自定义智能指针的时候，drop trait是手动编写的，而不是由编译器自动生成的。这是因为某些清理操作可能需要特定的顺序或逻辑，或者涉及到资源之间的依赖关系。通过手动实现 Drop trait，程序员可以控制清理操作的执行方式，确保资源被正确地释放和管理。

自定义智能指针

我们参照Box来实现一个简易版的自定义MyBox；

首先来看这样一段代码：

```rust
fn main() {
    let x = 5;
    let y = &x;
    assert_eq!(5,x);
    assert_eq!(5,*y);

  	let b = Box::new(5);
    assert_eq!(*b, 5);
}
```

三个assert语句都能通过； 常规的引用y通过解引用符号*

可以获得原始值5，这很容易理解，但奇怪的是，我们对Box使用*也可以获得原始值； 这是因为Box实现了上面提到的Deref trait ，在使用*时会自动调用deref函数,我们查阅源码可以看到：

```rust
impl<T: ?Sized, A: Allocator> const Deref for Box<T, A> {
    type Target = T;
    fn deref(&self) -> &T {
        &**self
    }
}
仿照它我们来完成MyBox：

use std::ops::Deref;
struct MyBox<T> {
    value: T
}

impl <T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox {
            value: x
        }
    }
}

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.value
    }
}

fn main() {
    let b = MyBox::new(5);
    let v1 = *b;
    let v2 = *(b.deref());  // 这样调用也是一样的
}这里很多人可能会有一点疑问，为什么官方Box的deref返回值是&**self而我们自定义的只是&self.value呢；这里要说一下，我们的MyBox只是一个简单的模仿，并没有Box开辟堆空间的能力而是把value存储在内存中；
```

查阅源码可以看到，官方Box的定义是：

pub struct Box<
    T: ?Sized,
    #[unstable(feature = "allocator_api", issue = "32838")] A: Allocator = Global,
>(Unique<T>, A);
在我们调用Box::new 时，系统会根据平台的不同去堆上开辟空间，然后Box持有唯一指针Unique（Unique并不会暴露给开发者使用）

有了这个前提，那么我们一步步分析解引用过程：

*self 解引用Box得到内部的 Unique<T>
**self 解引用Unique<T>，得到数据T
&**self 引用数据T
而我们的MyBox用value存储值，因此只返回&self.value就可以了；

根据智能指针存储方式的不同，有些还需要遵循Drop trait；这里我们实现的MyBox仅有一个常规分配的value，并不需要手动释放，因此并不需要显式实现Drop trait；