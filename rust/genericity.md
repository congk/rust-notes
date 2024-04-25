# 泛型

Rust 中对于泛型的定义与其他语言基本相同，但因为 Rust 定义结构体的方式与其他语言定义类、接口的方式不同，故实现过程可能有所差别。

泛型可以被用于结构体、方法、函数、枚举。

```rs
// 泛型可以被应用于结构体、函数、方法、枚举中，泛型定义数量没有限制
struct Point<T> {
    x: T,
    y: T,
}

// 结构体方法中使用泛型
impl<T> Point<T> {
    fn get_x(&self) -> &T {
        &self.x
    }

    // 泛型定义 U 仅在方法中定义
    fn test<U>(&self, value: U) -> U {
        value
    }
}


fn main() {
    let point = Point { x: 5, y: 6 };
    println!("point.x = {}", point.get_x());

    point.test(32);
}
```

针对特定具体类型单独定义方法，可以达到类似面向对象中的子类拓展和方法重载的效果。

```rs
// 仅对 Point<f64> 型数据增加 get_y 方法
impl Point<f64> {
    fn get_y(&self) -> f64 {
        return self.y;
    }
}

fn main() {
    let point = Point { x: 0.5, y: 0.6 };
    println!("point.y = {}", point.get_y());

    let point = Point { x: 1, y: 2};
    point.get_y();          // 编译报错，新的 point 类型为 Point<i32>，因此无法调用 get_y() 方法
}
```
