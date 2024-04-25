# 控制流运算符 match

match 表达式是一个可以用来处理枚举的控制流结构：它允许我们基于枚举拥有的变体来决定运行的代码分支，并允许代码通过匹配值来获取变体内的数据。

```rs
enum Coin {
  Penty,
  Nickel,
  Dime,
  Quarter,
  Other(u8)
}

fn value_in_cents(coin: Coin) -> u8 {
  match coin {
    // 模式 => 表达式
    Coin::Penty => {
      println!("Lucky penty!");
      1
    },
    Coin::Nickel => 5,
    Coin::Dime => 10,
    Coin::Quarter => 25,
    // 绑定值匹配
    Coin::Other(value) => value,
  }
}
```

`=>` 运算符用于将**模式匹配**与关联的**表达式**区分开，每个逻辑分支的表达式的结果会作为整个 match 表达式的结果返回。

## 匹配 Option<T>

```rs
fn plus_one(x: Option<i32>) -> Option<i32> {
  match x {
    None => None,
    Some(i) => Some(i + 1),
  }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

## 通配符 `_`

**match 表达式必须穷举所有可能性**，否则无法通过编译。但有时我们只关心部分模式匹配结果，那么可以使用通配符 `_` 来忽略其他情况。

> 类似 Typescript 中的 `switch` 语句的 `default`

```rs
let some_u8_value = 0u8;
match some_u8_value {
  1 => println!("one"),
  3 => println!("three"),
  _ => (),
}
```