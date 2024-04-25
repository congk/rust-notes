# 定义智能指针

通过实现 `Deref trait` 与 `Drop trait`，可以自行实现任意结构的智能指针。

```rs
// 导入 Deref trait
use std::ops::Deref;

// 使用元组结构定义
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

// 实现 Deref trait
impl<T> Deref for MyBox<T> {
    // type 语法用于定义关联类型
    type Target = T;

    // 实现 deref 以允许调用者通过 * 运算符访问值
    fn deref(&self) -> &Self::Target {
        &self.0
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);

    // *y 会被 Rust 隐式展开为 *(y.deref())
    assert_eq!(x, *y);
}
```

#### 函数与方法中的隐式解引用转换

**解引用转换（deref coercion）**是 Rust 为方法和函数的参数提供的一种便捷特性。当某个类型 T 实现了 Deref trait，并被当做参数传递给了函数或方法但与形参类型不一致时，Rust 会对 T 的引用不断尝试 Deref::deref 操作来获得与形参匹配的类型。该过程在编译阶段完成，所以解引用转换不会在运行时产生额外的性能开销。

#### 解引用转换和可变性

使用 Deref trait 能够重载不可变引用的 * 运算符，与之类似，使用 `DerefMut trait` 能够重载可变引用的 * 运算符。

Rust 会在类型与 trait 满足下面 3 个条件时执行解引用转换：

* 当 `T: Deref<Target=U>` 时，允许 `&T` 转换为 `&U`
* 当 `T: DerefMut<Target=U>` 时，允许 `&mut T` 转换为 `&mut U`
* 当 `T: Deref<Target=U>` 时，允许 `&mut T` 转换为 `&U`

对于第三种情况，Rust 允许将可变引用转为不可变引用，但无法将不可变引用转为可变引用（借用规则限制）。

#### 借助 Drop trait 在清理时运行代码

Drop trait 允许我们在变量离开作用域时执行某些自定义操作（不包括清理内存，Rust 会自动释放，不需要关注），而编译器则会自动将这些代码插入到合适的地方。

```rs
struct CustomSmartPointer {
    data: String
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data {}", self.data);
    }
}

fn main() {
    let a = CustomSmartPointer { data: String::from("a") };
    let b = CustomSmartPointer { data: String::from("b") };

    println!("CustomSmartPointer created.")
}
```

执行 `cargo run` 输出：

```sh
> cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.25s
     Running `target/debug/proj`
CustomSmartPointer created.
Dropping CustomSmartPointer with data b           # 变量的丢弃顺序与创建顺序是相反的，所以 b 会在 a 之前被丢弃。
Dropping CustomSmartPointer with data a
```

#### 使用 `std::mem::drop` 提前丢弃值

实现 Drop trait 类型的实例，其 drop 方法不允许显式调用，但是可以通过 `std::mem::drop` 方法来提前清理内存，该方法已预导入到了作用域内。

```rs
fn main() {
    let a = CustomSmartPointer { data: String::from("a") };

    // 主动调用会导致编译异常
    // a.drop();                  // 编译异常：explicit use of destructor method

    // 通过预导入的 drop 方法清理变量 a
    drop(a);

    println!("CustomSmartPointer dropped before the end of main.")
}
```

```sh
> cargo run
   Compiling proj v0.1.0 (/Users/coderk/works/rust/proj)
    Finished dev [unoptimized + debuginfo] target(s) in 0.15s
     Running `target/debug/proj`
Dropping CustomSmartPointer with data a           # 清理 a 的过程中会调用 a.drop()
CustomSmartPointer dropped before the end of main.
```

> 所有权系统会保证所有的引用有效，而 drop 只会被确定不再使用这个值时被调用一次。
