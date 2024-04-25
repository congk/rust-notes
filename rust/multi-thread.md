# 多线程

> * 并发编程（concurrent programming）：允许程序中的不同部分相互独立运行。
> * 并行编程（parallel programming）：允许程序中的不同部分同时执行。

## 使用线程同时运行代码

在大部分现代操作系统中，执行程序的代码会运行在进程（process）中，操作系统会同时管理多个进程。类似的，程序内部也可以拥有多个同时运行的独立部分，用来运行这些独立部分的叫做线程（thread）。

由于多个线程可以同时运行，所以将程序中的计算操作拆分至多个线程可以提高性能，但也增加了程序的复杂度，因为**不同的线程在执行过程中的具体顺序是无法确定的。**这可能导致一系列的问题：

* 当多个线程以不一致的顺序访问数据或资源时产生的竞争状态（race condition）。
* 当两个线程同时尝试获取对方持有的资源时产生的死锁（deadlock），它会导致两个线程无法继续运行。
* 只会出现在特定情形下且难以稳定重现和修复的 bug。

许多操作系统都提供了用于创建新线程的 API，这种**直接利用操作系统 API 来创建线程的模型称作 1:1 模型**。它意味着一个操作系统线程对应一个语言线程。

也有许多编程语言提供了自身特有的线程实现，这种由程序语言提供的线程常常被称为**绿色线程（green thread）**，使用绿色线程的语言会在拥有不同数量系统线程的环境下运行它们。**绿色线程也被称为 M:N 模型**，它表示 M 个绿色线程对应 N 个系统线程，M 与 N 不必相等。

由于绿色线程的 M:N 模型需要一个较大的运行时来管理线程，所以 **Rust 标准库仅提供了 1:1 线程模型的实现**。得益于 Rust 良好的底层抽象能力，Rust 社区中有许多支持 M:N 线程模型的三方包，你可以选择付出一定的性能开销来获得期望的特性，比如更强的线程控制能力、更低的线程上下文切换开销等。

## 使用 `spawn` 创建新线程

通过调用 `thread::spawn` 函数来创建线程，它接收一个闭包作为参数，该闭包会包含我们想要在新线程（生成线程）中运行的代码。

```rs
use std::{thread, time::Duration};

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            // thread::sleep(Duration::from_millis(1));
        }
    });

    for i in  1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

**主线程运行结束后，创建出的新线程就会相应停止，而不管它的打印任务是否完成**。

```
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

调用 `thread::sleep` 会强制当前线程停止运行一段时间，并允许一个不同的线程继续运行。这些线程可能会交替执行，但我们**无法对它们的执行顺序做出任何保证：执行顺序由操作系统的线程调度策略决定**。

## 使用 `join` 句柄等待所有线程结束

可以通过将 `thread::spawn` 返回的结果保存在一个变量中，来避免新线程出现不执行或不能完整执行的情况。`thread::spawn` 的返回值类型是一个自持有所有权的 `JoinHandle`，调用它的 `join` 方法可以**阻塞当前线程直到对应的新线程运行结束**。

```rs
use std::{thread, time::Duration};

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in  1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    // join 阻塞主线程，等待子线程执行结束
    handle.join().unwrap();

    println!("main thread end!");
}
```

## 在线程中使用 move 闭包

move 闭包尝尝被用来与 `thread::spawn` 函数配合使用，它**允许你在某个线程中使用来自另一个线程的数据**。

```rs
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    // move 关键字允许闭包捕获其定义作用于中值的所有权
    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v)
    });

    handle.join().unwrap();
}
```

**在新线程中使用主线程的值，必须使新线程获取其值的所有权而不能是借用**，否则 Rust 无法保证值在新线程中的有效性。
