# `Box<T>`

Rust 中最简单直接的一种智能指针，类型为 `Box<T>`，使我们可以将数据存储在堆上，并在栈中保留一个指向堆数据的指针。

和其他拥有所有权的值一样，Box 离开作用域时被释放，包括栈中的指针和指针指向的堆数据。

**使用场景**

* 当你又有一个无法在编译时确定大小的类型，但又不想在一个要求固定尺寸的上下文环境中使用这个类型的值时。
* 当你需要传递大量数据的所有权，但又不希望产生大量数据的复制行为时。
* 当你希望拥有一个实现了指定 trait 的类型值，但又不关心具体的类型时。

#### 使用 Box 定义递归类型

Rust 必须在编译时确定每一种类型占据的空间大小，但递归类型无法在编译时确定具体大小。递归类型的值可以在自身存储另一个相同类型的值，这种嵌套在理论上可以无穷尽下去，所以 Rust 无法计算出一个递归类型需要的具体空间。

```rs
// 为了计算一个枚举值所需要的空间，Rust 会遍历枚举中的每个成员来找到需要最大空间的那个变体。

enum List {           // 编译异常：recursive type `List` has infinite size
    Cons(i32, List),  // 嵌套导致 Rust 无法推断需要多大的空间来存储数据
    Nil,
}
```

`Box<T>` 是一个指针，指针的大小总是恒定的，它不会因为指向数据的大小而产生变化，因此可以利用 Box 将递归类型需要的大小固定下来。

```rs
enum List {
    // 新的 Cons 变体仅需要一部分存储 i32 的空间和一部分存储 Box 指针的空间
    Cons(i32, Box<List>),
    // Nil 没有值，因此它需要的存储空间比 Cons 小
    Nil,
}

use crate::List::*;

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
```

#### 将智能指针视作常规引用

`Box<T>` 属于智能指针的一种，因为它实现了 `Deref trait`，允许我们将 `Box<T>` 的值当做引用对待；它还实现了 `Drop trait`，因此当一个 `Box<T>` 值离开作用域时，Rust 会调用 `drop` 方法以清理指针指向的堆数据。

实现 `Deref trait` 意味着我们可以对其自定义解引用运算符`*`的行为，同时原本用于处理引用的代码，可以不加修改地用于处理智能指针。

```rs
#[test]
fn test() {
    let x = String::from("hello, world");
    let y = &x;
    assert_eq!(x, *y);

    // 使用 x.clone 是因为 Box::new() 会获取参数的所有权
    let y = Box::new(x.clone());
    assert_eq!(x, *y);
}
```