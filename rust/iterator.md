# 迭代器

**在 Rust 中，迭代器是惰性的（layzy）**。创建迭代器后，除非主动调用方法消耗和使用迭代器，否则它们不会产生任何的实际效果。

所有的迭代器都实现了标准库中的 Iterator trait，Iterator trait 提供了包含多种默认实现的方法。

```rs
trait Iterator {
    // type 定义迭代器关联类型，即元素类型
    type: Item;
    
    // Iterator trait 仅要求实现者实现 next 方法，返回 Option<Self::Item> 类型，迭代结束时返回 None
    fn next(&mut self) -> Option<Self::Item>;

    // ... 其他 Rust 默认实现
}
```

`next` 方法使用 `&mut self` 可变引用为参数，因为调用 `next` 方法会改变迭代器内部用来记录序列位置的状态，因此迭代器变量也应该是可变的。

```rs
#[test]
fn test() {
    let v1 = vec![1, 2, 3];

    // 声明可变变量
    let mut v1_iter = v1.iter();

    assert_eq!(v1_iter.next(), Some(&1));
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), Some(&3));
    assert_eq!(v1_iter.next(), None);
}
```

通过动态数组创建迭代器有 3 种方法：

* `iter()` 方法：生成的迭代器中，迭代器元素为动态数组元素值的不可变引用。
* `iter_mut()` 方法：生成的迭代器中，迭代器元素为动态数组元素值的可变引用。
* `iter_into()` 方法：生成迭代器并**获取动态数组所有权**。

#### 常用方法

`sum`：方法会**获取迭代器所有权**并反复调用 next 来遍历元素，对所有元素求和并将结果返回。

```rs
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();
// 这里 v1_iter 不需要声明 mut 是因为 sum 方法会直接获取所有权，不需要使用可变引用进行值修改
let total = v1_iter.sum();
```

`map`：接收一个用来处理所有元素的闭包作为参数，并生成一个新的迭代器

```rs
let v1 = vec![1, 2, 3];

// iter() 会生成迭代器，map 遍历所有迭代器元素并对每一项执行闭包调用，生成新的迭代器，与 JS 中的 Array.prototype.map 类似
let v1_iter = v1.iter().map(| x | x + 1);
```

`filter`：接收一个用来处理所有元素的闭包作为参数，遍历所有调用闭包，将闭包结果为 true 的元素生成新的迭代器

```rs
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter().filter(| x | x == &&1);

// 通过 collect() 方法消耗迭代器，将数据转换为指定类型，v2 是一个 Vec<&i32> 型动态数组
let v2: Vec<_> = v1_iter.collect();
```

#### 迭代器性能

尽管迭代器是一种高层次的抽象，但它在编译后生成了与手写 for 循环几乎一样的产物。

迭代器在 Rust 中是一种**零开销抽象（zero-cost abstraction）**，使用不会引入额外的运行时开销。
