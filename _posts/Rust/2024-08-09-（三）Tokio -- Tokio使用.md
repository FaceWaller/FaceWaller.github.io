Cargo.toml

```text
[package]
name = "studytokio"
version = "0.1.0"
edition = "2024"

[dependencies]
tokio = { version = "1", features = ["full"] }
```

main.rs

```text
use std::{thread, time::Duration};



#[tokio::main] // 改为异步入口

async fn main() {

// 异步执行

let t = say_tokio();

println!("{:?} Hello, ", thread::current());

t.await;



// 异步切换线程

let handle = tokio::spawn(async {

println!("{:?} spawned hello tokio", thread::current());

});

handle.await.unwrap();



tokio::time::sleep(Duration::from_secs(10)).await;

}



async fn say_tokio() {

println!("Tokio!");

}
```

这里列举两个简单使用的例子，不多说明；