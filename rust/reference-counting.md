# `Rc<T>`

Rust 通过 `Rc<T>` 类型来支持多重所有权，Rc 是 Reference Counting（引用计数）的缩写。`Rc<T>` 类型实例内部维护一个用于记录引用值引用计数的计时器，从而确认这个值是否仍在使用。如果对一个值的引用次数为 0，这个值可以被安全的清理而不需要担心引用失效的问题。

当你希望将堆上的数据分享给程序的多个部分同时使用，而又无法在编译器确认哪个部分会最后释放这些数据时，就可以使用 `Rc<T>` 类型处理。

> 注意：`Rc<T>` **只能被用于单线程场景**，它通过**不可变引用**使你可以在程序的不同部分之间**共享只读数据**，多个指向同一数据的可变引用会导致数据竞争和数据不一致，不符合借用规则。

```rs
// Rc 并未预导入，需要显式导入作用域
use std::rc::Rc;

struct Dog {
    name: String,
}

fn main() {
    let dog = Dog {
        name: String::from("Dog"),
    };

    // Rc::new 或获取 rect 所有权，并返回 `Rc<T>` 实例
    let a = Rc::new(dog);

    // Rc::clone 会复制智能指针，并将引用计数 + 1
    let b = Rc::clone(&a);
    let c = Rc::clone(&a);

    // a,b,c 将共享 dog 数据的所有权
    println!("{}, {}, {}", a.name, b.name, c.name)
}
```

`Rc::clone` 不会执行数据深拷贝，只会增加引用技术，这与绝大多数类型实现的 clone 方法有明显不同。同样的，`Rc<T>` 的 Drop trait 实现会在 `Rc<T>` 类型变量离开作用域时自动将引用技术减 1。

## 强引用与弱引用

`Rc<T>` 类型内部存在 `strong_count()` 与 `weak_count()` 两个计数查询方法，分别用于获取强引用计数与弱引用计数。

```rs
use std::rc::Rc;

struct Dog {
    name: String,
}

fn main() {
    // 获取 Dog 实例数据所有权并生成强引用实例，strong_count = 1
    let a = Rc::new(Dog {
        name: String::from("Dog"),
    });
    // 强引用拷贝，b 类型为 Rc<Dog>，strong_count + 1
    let b = Rc::clone(&a);
    // 弱引用拷贝，c 类型为 Weak<Dog>，weak_count + 1
    let c = Rc::downgrade(&a);

    println!("strong_count: {}", Rc::strong_count(&a));     // output: strong_count: 2
    println!("weak_count: {}", Rc::weak_count(&a));         // output: weak_count: 1
}
```

* `Rc::clone()` 方法返回的是 `Rc<T>` 类型实例，是值的强引用，拷贝会使 `strong_count` 计数 +1；
* `Rc::downgrade()` 方法将返回 `Weak<T>` 类型实例，是一个弱引用，拷贝会使 `weak_count` 计数 +1；

弱引用相较于强引用的区别在于，**弱引用不会表达所有权关系**。`Rc<T>` 并不会在清理数据前要求 `weak_count` 必须为 0。

**弱引用主要用来解决循环引用造成的内存泄露问题**，一旦强引用计数为 0，任何由弱引用组成的引用循环都会被打破。
