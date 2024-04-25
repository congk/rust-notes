# 使用消息传递在线程间转移数据

> 不要通过共享内存来通信，而是通过通信来共享内存。

Rust 在标准库中实现了一个名为通道（channel）的编程概念，它可以被用来实现基于消息传递的并发机制。通道由发送者（transmitter）和接收者（receiver）组成，通过调用发送者的方法来传送数据，并通过检查接收者来获取数据。当丢弃了发送者或接收者的任何一端，我们称响应的通道被关闭了。

Rust 中通过 `mpsc::channel` 方法创建通道，mpsc 是”multiple producer, single consumer“（多个生产者，单个消费者）的缩写。Rust 标准库中特定的实现方式使得通道可以拥有多个生产内容的发送端，但只能拥有一个消耗内容的接收端。

```rs
use std::{thread, sync::mpsc};

fn main() {
    let (tx, rx) = mpsc::channel();

    // 通过 move 关键字将 tx 发送端所有权交给新线程，主线程保留 rx 接收端，tx 与 rx 构成一条单工链路
    thread::spawn(move || {
        let val = String::from("hi");

        // send 返回 Result<T, E> 型数据，若接收端已被丢弃，会返回相应 Err 信息
        tx.send(val).unwrap();
    });

    // recv() 接收数据会阻塞主线程的执行，直到有值被传入
    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

**新线程必须拥有发送端的发有权才能通过通道来发送消息**。

通道的接收端有两个可用于获取消息的方法：`recv` 和 `try_recv`。

* `recv` 会阻塞线程的执行，直到有值被传入通道。一旦有值被传入通道，`recv` 会将它包裹在 `Result<T, E>` 中返回。
* `try_recv` 方法不会阻塞线程，它会立即返回 `Result<T, E>`: 当通道中存在消息时，返回包含该消息的 Ok 变体；否则返回 Err 变体。

**`send` 函数会获取参数的所有权，并在参数传递时将所有权转移给接收者**

## 发送多个值并观察接收者的等待过程

```rs
use std::{thread, sync::mpsc, time::Duration};

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let v = vec![1, 2, 3, 4, 5];

        for i in v {
            tx.send(i).unwrap();
            thread::sleep(Duration::from_millis(1000))
        }
    });

    for received in rx {
        println!("Got: {}!", received);
    }
}
```

在主线程中，rx 被视作**迭代器**，而不再显式地调用 recv 函数。迭代中的代码会打印出每个接收的值，并在通道关闭时退出循环。

## 通过克隆发送者创建多个生产者

```rs
use std::{thread, sync::mpsc, time::Duration};

fn main() {
    let (tx, rx) = mpsc::channel();

    // 通过 `mpsc::Sender::clone` 克隆 tx
    let tx1 = mpsc::Sender::clone(&tx);

    thread::spawn(move || {
        let v = vec![1, 2, 3, 4, 5];
        for i in v {
            tx.send(i).unwrap();
            thread::sleep(Duration::from_millis(1));
        }
    });
    thread::spawn(move || {
        let v = vec![6, 7, 8, 9, 10];
        for i in v {
            tx1.send(i).unwrap();
            thread::sleep(Duration::from_millis(1));
        }
    });

    for received in rx {
        println!("Got: {}!", received);
    }
}
```

因为**无法保证线程的执行顺序**，所以打印顺序会因操作系统而异。
