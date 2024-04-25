# 编写测试代码

Rust 在语言层面内置了编写测试代码，执行自动化测试任务的功能。Rust 语言中测试是一个函数，它被用于验证非测试代码是否按照期望的方式执行。

Rust 中的测试就是一个有 test 注解属性的函数。**属性（attribute）** 是一种用于修饰 Rust 代码的元数据。

```rs
fn main() {}

// test 注解，标记该函数是一个测试用例
#[test]
    fn case_1() {
        // assert_eq! 宏断言
        assert_eq!(1 + 1, 2);
    }

// 向 cargo test 传递测试名称，便于单独执行某一个测试用例
#[cfg(test)]
// 将测试函数放到某个模块内，测试模块与普通模块没有任何区别
mod tests {
    // 注解 test
    #[test]
    fn case_2() {
        assert_eq!(2 + 2, 4);
    }
}

```

定义完测试函数，可以通过 `cargo test` 进行自动化测试。

```sh
> cargo test
   Compiling rust-package v0.1.0 (/Users/coderk/works/rust-package)
    Finished test [unoptimized + debuginfo] target(s) in 0.13s
     Running unittests src/main.rs (target/debug/deps/rust_package-f76dd92c6919a51f)

running 2 tests
test case_1 ... ok
test tests::case_2 ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

**除测试函数之外，Rust 还能够编译在 [API 文档](rust/apidoc.md)中出现的任何代码示例！**

#### 常用断言

| 宏 | 说明 |
|:----|:-----|
| `assert!` | 当值为 `false`，`assert!` 宏会调用 `panic!` 宏导致测试失败 |
| `assert_eq!` | 要求两个参数相等，否则会调用 `panic!` 宏，并打印两个参数的值 |
| `assert_ne!` | 要求两个参数不等，否则会调用 `panic!` 宏，并打印两个参数的值 |

任何在 `assert!`、`assert_eq!`、`assert_ne!` 必要参数之后的参数，都会被一起传递给 `format!` 宏，以此自定义测试失败时的错误提示信息。

```rs
#[test]
fn test() {
    let x = 1;
    let y = 1;
    
    assert!(x == y);      // 测试通过
    assert_eq!(x, y);     // 测试通过
    assert_ne!(x, y);     // 测试失败

    // 自定义错误信息
    assert!(x != y, "value of x is {}, value of y is {}", x, y);
}
```

#### 使用 should_panic 检查 panic

标记了 `should_panic` 属性的测试函数，会在发生 panic 时顺利通过测试，而不发生 panic 会导致测试失败。

```rs
#[test]
#[should_panic]             // 标注 should_panic
fn test() {
    panic!("Ouch!!!");      // 发生 panic，该测试用例通过测试
}
```

通过 `should_panic(expected="")` 准确定位错误信息中**包含**指定字符串的 panic，否则使测试失败。

```rs
#[test]
#[should_panic(expected="Ouch")]          // 标注 should_panic
fn test2() {
    panic!("No!!!");                      // 发生 panic，但错误信息不包含 `Ouch`，所以测试失败
}
```

#### 使用 `Result<T, E>` 编写测试

使用 Result 枚举编写测试时，不再调用 `assert!`、`assert_eq!`、`assert_ne!`，而是将测试函数定义返回值为 Result，Ok 变体为表示测试通过。

测试函数内部不能调用 `panic!`，也不能对函数添加 should_panic 注解，而应该使用 Err 变体。

```rs
#[test]
fn test() -> Result<(), String> {
    if true {
        Ok(())
    } else {
        Err(String::from("Ouch!!!"))
    }
}
```