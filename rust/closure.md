# 闭包

Rust 中的闭包是一种可以存入变量或作为参数传递给其他函数的匿名函数；每个闭包都有其独立的匿名类型，即便闭包签名相同，其类型也不同。

与 fn 定义的函数不同，闭包并不强制要求你标准参数和返回值类型，编译器能够可靠地推断出闭包参数和返回值类型。但是，**当编译器接收第一次入参并推导出类型信息后，会将闭包的类型信息绑定在闭包中**，再次对闭包传递非匹配类型参数会引发编译异常。

```rs
let example_closure = |x| x;

// 第一次调用闭包，编译器推导闭包类型为 Fn(String) -> String，并将类型信息保存
let s = example_closure(String::from("hello"));

// 第二次调用参数使用 i32 型数据，会引发编译异常
let n = example_closure(5);
```

**闭包可以从定义它的作用域中捕获值，但通过 `fn` 定义的函数不行。**

```rs
let x = 1;

fn equal_to_x(y: i32) { x ==  y }     // 编译异常，fn 函数无法访问 x

let equal_to_x = | y | x == y;        // 编译通过，闭包可以从定义它的作用域内捕获值
```

闭包从环境中捕获值的方式有三种：`Fn`、`FnMut`、`FnOnce`

**1. `Fn trait`：从作用域环境中获取不可变的借用值**

```rs
let x = String::from("x");

// Fn(&str) -> bool 类型，&x 是一个不可变引用
let contains_x = | y: &str | y.contains(&x); 
```

**2. `FnMut trait`：从作用域中获取可变借用值**

```rs
let mut x = String::from("x");

// FnMut(&str) -> () 类型，闭包内允许修改 x 的值
let mut append_to_x = | y: &str | x.push_str(y);
append_to_x("y");

println!("{}", x);    // output: xy
```

**3. `FnOnce trait`：从作用域中夺取所有权**

```rs
let mut x = String::from("x");

// FnOnce(&str) -> () 类型，闭包会夺取 x 的所有权并修改 x 的值；定义 FnOnce 闭包需要使用 move 关键字
let mut append_to_x = move | y: &str | x.push_str(y);
append_to_x("y");

println!("{}", x);    // 编译异常
```

实现 `FnOnce trait` 特征的闭包**夺取所有权行为发生在闭包定义时**，与闭包是否被调用无关。