# 处理错误

Rust 将错误区分设计 `Result<T, E>` 与终止程序运行的 `panic!` 宏。

## `panic!`

程序会在 `panic!` 执行时打印出一段错误提示信息，展开并清理当前的调用栈，然后退出程序；Rust 的设计中 `panic!` 无法被其他语言中类似 `try...catch` 语句捕获。

>
#### panic 的栈展开与终止

当 panic 发生时，程序会**默认**开始栈展开，意味着 Rust 会沿着调用栈反向顺序遍历所有调用函数，并依次清理其中的数据。为了支持这种遍历和清理操作，需要在二进制中存储更多额外信息。

除了展开，也可以通过修改 `Cargo.toml` 中的 [profile] 区域添加 `panic = 'abort'` 直接结束程序而不进行任何清理工作，程序使用过的内存只能由操作系统进行回收。

```
[profile.release]
panic = 'abort'
```

#### 回溯信息

通过 `RUST_BACKTRACE=1` 启动程序，当 panic 发生时，控制台会打印出回溯列表，包含所有的调用函数列表，类似 JS 中的 `stack` 信息。

```rs
fn main() {
    let v = vec![1, 2];
    // 索引超出了动态数组长度，会发生 panic
    println!("{}", &v[100]);
}
```


```sh
> cargo run
  Finished dev [unoptimized + debuginfo] target(s) in 0.00s
    Running `target/debug/rust-package`
thread 'main' panicked at 'index out of bounds: the len is 2 but the index is 100', src/main.rs:4:21
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace

> RUST_BACKTRACE=1 cargo run
  Finished dev [unoptimized + debuginfo] target(s) in 0.00s
    Running `target/debug/rust-package`
thread 'main' panicked at 'index out of bounds: the len is 2 but the index is 100', src/main.rs:4:21
stack backtrace:
   0: rust_begin_unwind
             at /rustc/8460ca823e8367a30dda430efda790588b8c84d3/library/std/src/panicking.rs:575:5
   1: core::panicking::panic_fmt
             at /rustc/8460ca823e8367a30dda430efda790588b8c84d3/library/core/src/panicking.rs:64:14
   2: core::panicking::panic_bounds_check
             at /rustc/8460ca823e8367a30dda430efda790588b8c84d3/library/core/src/panicking.rs:159:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/8460ca823e8367a30dda430efda790588b8c84d3/library/core/src/slice/index.rs:260:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/8460ca823e8367a30dda430efda790588b8c84d3/library/core/src/slice/index.rs:18:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at /rustc/8460ca823e8367a30dda430efda790588b8c84d3/library/alloc/src/vec/mod.rs:2732:9
   6: rust_package::main
             at ./src/main.rs:4:21
   7: core::ops::function::FnOnce::call_once
             at /rustc/8460ca823e8367a30dda430efda790588b8c84d3/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

## Result 源码解读

```rs
// Result 是一个枚举型，包含 Ok 与 Err 两个变体
enum Result<T, E> {
  Ok(T),
  Err(E),
}

// 常用方法
impl<T, E> Result<T, E> {

  // 当值为 Ok 时，返回 Ok 内部的值，否则执行 `panic!` 宏
  fn unwrap(&self) -> T {}

  // 与 unwrap 类似，支持自定义 panic! 宏的 msg 信息
  fn expect(&self, msg: &str) -> T {}
}
```

**通过 `?` 运算符传递 Err**

> `?` 运算符是一个语法糖，仅能被用于返回 `Result<T, E>` 的函数中

```rs
use std::fs::File;
use std::io::{Error, Read};

fn read_username_from_file() -> Result<String, Error> {
    //  File::open() 方法返回一个 Result<File> 枚举
    // ? 运算符标识若值为 Ok(File)，则将值 File 赋值给 f 变量；若值为 Err，终止当前函数并将 Err 返回函数调用者
    let mut f = File::open("hello.txt")?;

    // 声明可变变量 String 用于存储
    let mut s = String::new();

    // 将文件内容读取到 s 中，句末的 ? 运算符确保在读取过程中若发生异常则终止函数，并将 Err 返回给函数调用者
    f.read_to_string(&mut s)?;

    // 返回 Ok 变体并携带值
    Ok(s)
}

fn main() {
    // read_username_from_file 执行过程中出现的 Err 会直接返回至函数调用方
    match read_username_from_file() {
        Ok(str) => println!("{}", &str),
        Err(_) => println!("Read file error"),
    }
}
```