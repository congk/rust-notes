# 单元测试与集成测试

#### 单元测试

Rust 中，一般将单元测试代码与需要测试的代码放在 src 的同一个目录下的同一文件中，约定每个文件中新建一个 tests 模块存在测试函数，并使用 cfg(test) 对模块进行标注。

标注 `#[cfg(test)]` 可以让 Rust 只在执行 `cargo test` 命令时编译和运行该部分测试代码，并在 `cargo build` 时剔除它们。

```rs
pub fn add_two(a: i32) -> i32 {
    a + 2
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    // 将 crate 内容导入作用域
    use super::*;

    #[test]
    fn internal() {
        // Rust 允许测试私有私有函数
        assert_eq!(4, internal_adder(2, 2));
    }
}
```

#### 集成测试

Rust 约定集成测试目录为 `tests` 目录，处于项目根目录中，与 `src` 同级，只有在执行 `cargo test` 命令时，Cargo 才会对该目录文件进行编译，自动查找目录下的集成测试文件（不包含子目录文件）。

tests 目录下可以创建任意多的测试文件，Cargo 会在编译时将每个文件处理成一个 binary crate。

tests 目录下的任何代码不需要添加 `#[cfg(test)]` 注解。

通过 `cargo test --test <filename>` 来指定特定的集成测试用例。

tests 子目录中的文件不会被视作单独的集成测试文件进行编译。

集成测试仅能针对 library crate 包进行，即工程中必须存在 `src/lib.rs` 文件，否则无法通过 tests 目录定义集成测试。
