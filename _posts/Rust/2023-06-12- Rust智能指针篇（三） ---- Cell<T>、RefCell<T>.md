---
layout: post                        ## 文章使用的模板
title: Rust智能指针篇（三） ---- Cell<T>、RefCell<T>				## 文章的标题
date: 2023-06-12						## 发布时间
author: "FaceWaller"                ## 作者
categories: Rust
tags: Rust
---

内部可变性

这是一个反直觉的特性，它允许你即使在使用不可变引用时也可以改变数据；

通常来说，变量都是通过mut关键字修饰实现的，比如

let var: T     // 那么var就是不可变的，而且var内部的成员也都是不可变的；

let mut var : T   // 那么var就是可变的；
但如果使用Cell、RefCell包装，这样就可以修改它了，这就是内部可变性

Cell和RefCell只适用于单线程场景

区别

Cell和RefCell都可以实现内部可变性，但有些区别：

Cell 是一个简单的类型，可以存储一个单一的值。它可以在不使用可变引用的情况下，对其内部的值进行修改。但是， Cell 只能存储实现了 Copy trait的类型。

RefCell 是一个更复杂的类型，可以在运行时进行借用检查。与 Cell 不同， RefCell 可以存储不可复制的类型。但是， RefCell 在运行时检查借用规则，如果违反了规则，就会导致程序崩溃。

使用场景

假设有这样一个处理信息的特征，以及两个遵守特征的结构体：

```rust
trait SourceHandleTrait {
    fn handle(&self, msg: String);
}

struct SourceHandle1 {}
impl SourceHandleTrait for SourceHandle1 {
    fn handle(&self, msg: String) {
        println!("handle1 handle {}", msg); // handle1 收到信息后执行打印
    }
}

struct SourceHandle2 {
    msgs: Vec<String>
}

impl SourceHandleTrait for SourceHandle2 {
    fn handle(&self, msg: String) {
        self.msgs.push(msg);		// handle2 收到消息后存到数组中
    }
}
由于特征中的函数参数 &self 是不可变的，内部的字段也不可变，因此 self.msgs.push(msg); 这一行会报错;

对此，我们的修改思路有两种，一种是修改特征参数为 &mut self，另外一种我们可以通过RefCell实现

struct SourceHandle2 {
    msgs: RefCell<Vec<String>>
}

impl SourceHandleTrait for SourceHandle2 {
    fn handle(&self, msg: String) {
        self.msgs.borrow_mut().push(msg)
    }
}
```

通过RefCell修饰，即使没有mut，仍然可以修改msgs；使用borrow或者borrow_mut去获取它包装的值，这里因为要对数组进行修改，因此我们使用borrow_mut

完整代码如下：
```rust
use std::cell::RefCell;
trait SourceHandleTrait {
    fn handle(&self, msg: String);
}

struct SourceHandle1 {

}
impl SourceHandleTrait for SourceHandle1 {
    fn handle(&self, msg: String) {
        println!("handle1 handle {}", msg); // handle1 收到信息后执行打印
    }
}

struct SourceHandle2 {
    msgs: RefCell<Vec<String>>
}

impl SourceHandleTrait for SourceHandle2 {
    fn handle(&self, msg: String) {
        self.msgs.borrow_mut().push(msg); // handle2 收到消息后存到数组中
    }
}

#[test]
fn handle_source() {
    let str1 = "source string1".to_string();
    let s1 = SourceHandle1{};
    s1.handle(str1);

		let str2 = "source string2".to_string();
		let s2 = SourceHandle2{
    		msgs: RefCell::new(vec![])
		};
		s2.handle(str2);
}
```

Rc与RefCell结合

rc的特点是允许有多个所有者，RefCell是允许修改数据，将他们结合起来就可以实现多个可变数据所有者的需求；

假设有这样一个场景，两份表格里面包含了同一个人的信息，我们希望在修改这个人的信息时，两份表格能够自动跟随变化；这时就可以考虑将Rc与RefCell结合使用，代码如下：

```rust
use std::{cell::RefCell, rc::Rc};

#[derive(Debug)]
struct Number {
    id: i32,
    p: Rc<Personal>
}

#[derive(Debug, Clone)]
struct Personal {
    age: RefCell<i32>,
    name: String
}

#[test]
fn handle_source() {
let person = Personal {
    age: RefCell::new(10),
    name: "alan".to_string()
};
let rc_personal = Rc::new(person);

let n1 = Number {
    id: 1001,
    p: rc_personal.clone()
};

let n2 = Number {
    id: 2002,
    p: rc_personal.clone()
};

println!("person {:?}", rc_personal);
println!("n1 {:?}", n1);
println!("n2 {:?}", n2);

*rc_personal.clone().age.borrow_mut() = 20; 
println!("============= 修改年龄 =============");

println!("person {:?}", rc_personal);
println!("n1 {:?}", n1);
println!("n2 {:?}", n2);
```

}
首先我们定义了表格struct，其中包含一个p字段，类型为Rc包装的Personal，有了Rc的包装就可以保证多个结构体都可以持用同一个Personal; 接下来是Personal结构体，我们假定age是要变化的所以age的类型是用RefCell包装的；

测试代码中我们生成了两个表格n1、n2持有同一个personal，然后再修改personal的age，通过打印发现，n1、n2中的值也会跟着变化；

思考

可以看到内部可变性突破了很多rust的限制，通过这个特性用rust实现一个链表也变的容易起来；

同时就像unsafe代码块一样，使用了RefCell就意味着静态编译检查不再保证程序的安全，要由开发者自己负责安全情况，这就意味着是有一定风险的；

在实际项目中，就我来说，也没有用到过RefCell，使用RefCell的场景基本都可以通过代码设计来规避，毕竟它并不是那么的安全；