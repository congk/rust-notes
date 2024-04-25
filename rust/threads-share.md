# 共享状态的并发

基于共享内存的并发通信机制类似于多重所有权概念：多个线程可以同时访问相同的内存地址。

## 互斥体一次只允许一个线程访问数据

互斥体（mutex）是 mutual exclusion 的缩写。一个互斥体在任意时刻值允许一个线程访问数据。为了访问互斥体中的数据，线程必须首先发出信号来获取互斥体的锁。锁作为互斥体的一部分，这种数据结构被用来记录当前谁拥有数据的唯一访问权。

互斥体出了名的难用，因为必须牢记两条规则：

* 必须在使用数据前尝试获取锁。
* 必须在使用完互斥体守护的数据后释放锁，这样其他线程才能继续完成获取锁的操作。

## `Mutex<T>`

```rs
use std::sync::Mutex;

fn main() {
    // 声明互斥体
    let m = Mutex::new(5);
    {
        // 为了访问 Mutex<T> 中的数据，需要首先调用它的 lock 方法来获取锁，以访问互斥器中的数据
        let mut num = m.lock().unwrap();
        // 一旦获取了锁，便可以将其返回值视作一个可变引用
        *num = 6;
        // 离开作用域，锁被自动释放
    }
    println!("m = {:?}", m);
}
```

为了访问 `Mutex<T>` 实例中的数据，首先需要调用 `lock` 方法来获取锁。这个调用会**阻塞当前线程**直到线程获取到锁为止。

当前线程对于 `lock` 函数的调用，会在某个持有锁的线程发生 panic 时失败并返回 Err 变体。

`lock()` 方法获取到锁后，会调用返回 `Result<T, E>` 包裹的 `MutexGuard` 智能指针，其实现了 `Reref trait`，因此可以将其视作一个可变引用；同时其实现了 `Drop trait`，以在其离开作用域时**自动释放锁**

## 多个线程间共享 `Mutex<T>`（多线程的多重所有权）

`Rc<T>` 允许值具有多重所有权，但在跨线程使用时并不安全，仅限于**单线程**场景内使用。

`Arc<T>` 与 `Rc<T>` 类似，它是 Rust 提供的另一个**允许多重所有权的智能指针，且能够保证在多线程场景下的线程安全，但相应花费更多的性能开销**。

```rs
use std::{
    sync::{Arc, Mutex},
    thread,
};

fn main() {
    // 声明计数，通过 Arc 来保证多重所有权的线程安全，同时通过 Mutex 确保同一时刻仅有一个线程具备唯一访问权
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 1..10 {
        // 通过 Arc::clone 克隆 counter，向每个线程转移所有权
        let counter = Arc::clone(&counter);
        // 通过 move 关键字将克隆的 counter 所有权转移至新线程
        let handle = thread::spawn(move || {
            // 线程内获取互斥锁，以便于统计计数
            let mut val = counter.lock().unwrap();
            // val 是 MutexGuard<i32> 类型智能指针，可被视为一个可变引用
            *val += 1;
        });

        handles.push(handle);
    }

    for handle in handles {
        // 确保所有线程执行完毕
        handle.join().unwrap();
    }

    // 主线程获取锁以取值
    println!("Result: {}!", counter.lock().unwrap());
}
```

虽然 `counter` 是不可变的，但我们仍然能获取到其内部值的可变引用，这意味着，`Mutex<T>` 与 `RefCell<T>` 相似，`Mutex<T>` 也提供了**内部可变性**。

#### 使用 `Mutex<T>` 造成的死锁

使用 `Mutex<T>` 也会产生死锁的风险：当某个操作需要同时锁住两个资源，而两个线程分别持有一个锁并相互请求另一个锁时，这两个线程就会陷入无穷尽的等待过程。