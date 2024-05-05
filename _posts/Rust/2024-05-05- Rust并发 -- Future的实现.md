# Rust 并发 ---- Future的实现

 通过async 函数或者 async 块都可以获得一个 Future， 然后通过 await 的方式去调用

```rust
let ft1 = async_foo();
let ft2 = async {};
```

但我们并不清楚 Future 是如何产生的，怎么被 executor 处理的；接下来我们就深入研究下 Future 的内部实现

### Future

```rust
pub enum Poll<T> {
    Ready(T),
    Pending,
}

pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```



这个 trait 看起来并不复杂，Output 指定了一个返回值的类型，具体的类型会在实现具体的 trait 时确定；

poll 函数则是异步进行的关键，返回值是一个枚举，分别代表完成和等待状态；可参数却不好理解，Pin 和 Context 分别代表什么，是什么作用？理解了它们也就理解了 Future 的关键；



### Context

```rust
pub struct Context<'a> {
    waker: &'a Waker,
    _marker: PhantomData<fn(&'a ()) -> &'a ()>,
    _marker2: PhantomData<*mut ()>,
}
```

点进去会发现 Context 是对 Waker 的封装，而 Waker 是对RawWaker的封装，RawWaker又是对RawWakerVTable的封装

```rust
pub struct RawWakerVTable {
    clone: unsafe fn(*const ()) -> RawWaker,
    wake: unsafe fn(*const ()),
    wake_by_ref: unsafe fn(*const ()),
    drop: unsafe fn(*const ()),
}
```

最后看到 RawWakerVTable 的成员变量都是 unsafe *const 类型；再往下，似乎一切线索都断了，定义的变量中核心似乎只有一个 wake；

实际上，Rust 标准库本身并不提供异步运行时，只是提供基本的接口，具体的实现在三方库中（比如 tiokio）, 我们在查看 tokio 库的时候，就会看到类似这样的代码：

```rust
static WAKER_VTABLE: RawWakerVTable =
    RawWakerVTable::new(clone_waker, wake_by_val, wake_by_ref, drop_waker);
```



我们都知道调度器通过调用 poll 的方法来让 future 往下执行，如果返回 Poll::Pending 就会阻塞 Future，之后在合适的时间再通过 wake 把 future 唤醒继续执行，而 Context 的作用正是被唤醒；

然而可惜的是，想要再进一步了解完整的唤醒机制，还需要阅读三方库，这里就不继续展开了😮‍💨

### Pin

##### Pin 是什么

Pin 是一个标准库中的类型，它的作用是确保一个值在内存中的位置不会改变；在处理自引用类型时会用到这条特性，举个例子

```rust
use std::mem;

#[derive(Debug)]
struct ReferentailSelf {
    a: String,
    b: *const String,
}

impl ReferentailSelf {
    fn new(txt: &str) -> Self {
        ReferentailSelf {
            a: String::from(txt),
            b: std::ptr::null(),
        }
    }

    fn init(&mut self) {
        let self_ref: *const String = &self.a;
        self.b = self_ref;
    }
}

fn main() {
    let mut s1 = ReferentailSelf::new("111");
    s1.init();
    let mut s2 = ReferentailSelf::new("222");
    s2.init();

    println!("s1.b {}", unsafe {&*s1.b});
    println!("s2.b {}", unsafe {&*s2.b});

    mem::swap(&mut s1, &mut s2);

    println!("s1.b {}", unsafe {&*s1.b});
    println!("s2.b {}", unsafe {&*s2.b});
  

}

/*
输出内容
	s1.b 111
	s2.b 222
	s1.b 111
	s2.b 222
*/
```

 我们定义了一个结构体ReferentailSelf，其中 b 是一个裸指针类型，指向 a，在我们交换了 s1 和 s2 之后，会发现 b 的值并没有改变；

这是因为 swap 函数用来交换变量的指向，原本 s1.a 指向堆上的 "111"  s2.a 指向堆上的 "222", 交换后s1.a --> "222", s2.1--> "111", 同时 原本 s1.b --> s1.a 变成了 s1.b --> s2.a， s2.b-->s2.a 变成了 s2.b->s1.a;



交换前：

<img src="https://github.com/FaceWaller/blogImages/blob/master/rust/swap1.png?raw=true" alt="swappng" style="zoom:40%;" />

交换后：

<img src="https://github.com/FaceWaller/blogImages/blob/master/rust/swap2.png?raw=true" alt="swappng" style="zoom:40%;" />

这并不是我们所期望的，甚至还会出现生命周期不一致等更严重的问题；

关于 Pin 的详解，我认为这篇文章特别细致全面，非常值得看一遍，最好把里面的例子都敲一遍加深理解

> https://rustcc.cn/article?id=1d0a46fa-da56-40ae-bb4e-fe1b85f68751

##### Future 与 Pin

再回到 future， 为什么 poll中的参数也用了Pin<&mut Self>呢?

实际就是async fn生成的Future，往往都是自引用的；

假设有这样一段代码

```rust
async fn writeFile() -> Result<(), ()> {
    let file = fs::File::create("test").await?;
    file.write_all(b"hello world!").await?;
    Ok(())
}
```

进行编译时，会把他变成一个形似 writeFileFuture的结构，大概如：

```rust

enum WriteFileFuture {
    Init,
    CreateFile1(fs::File),
    CreateFile2(fs::File),
    Done,
}

impl Future for WriteFileFuture {
    type Output = Result<(), std::io::Error>;
    fn poll(self: std::pin::Pin<&mut Self>, cx: &mut std::task::Context<'_>) -> std::task::Poll<Self::Output> {
        let this = self.get_mut();
        loop {
            match this {
                WriteFileFuture::Init => {
                    let fnt = fs::File::create("test");
                    match fnt.poll(cx) {
                        Poll::Ready(file) => {
                            *self = WriteFileFuture::CreateFile1(file);
                        }
                        Poll::Pending => {
                            return Poll::Pending;
                        }
                    } 
                }
                WriteFileFuture::CreateFile1(file) => {
                    let fnt1 = file.write_all(b"hello world2!\n");
                    match fnt1.poll(cx) {
                        Poll::Ready(file) => {
                            *self = WriteFileFuture::CreateFile2(file);
                        }
                        Poll::Pending => {
                            return Poll::Pending;
                        }
                    } 
                }
                WriteFileFuture::CreateFile2(file) => {
                    let fnt2 = file.write_all(b"hello world1!\n");
                    match fnt2.poll(cx) {
                        Poll::Ready(file) => {
                            *self = WriteFileFuture::Done;
                        }
                        Poll::Pending => {
                            return Poll::Pending;
                        }
                    } 
                }
                WriteFileFuture::Done => {
                    return Poll::Ready(Ok(()));
                }
            }
        }
    }
}
```

但这里会有个问题，file未必支持clone，那file的所有权该归谁，fnt1还是fnt2，还是当前的WriteFileFuture，实际上编辑器会给给WriteFileFuture生成一个属性，其他包含 fn1  fn2  file 类似：

```rust
enum WriteFileFuture {
    Init,
    CreateFile1(WriteFileFutureParam),
    CreateFile2(WriteFileFutureParam),
    Done,
}

struct WriteFileFutureParam {
    file,
    fnt1,
    fnt2,
}
```

此时 fn1和fnt2也是需要file的， 这就有了前面所有的自引用模式， 在loop循环中，为了避免WriteFileFuture移动而导致的自引用异常，因此就用到了Pin；

实际上，Pin的初衷就是为了解决future的自引用；
