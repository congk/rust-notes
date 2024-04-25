# 控制测试的运行方式

```sh
# 指定命令行参数
cargo test --help

# 指定待测试二进制文件的运行参数
cargo test -- --help
```

#### 控制测试流程

Rust 默认使用多线程并行执行测试用例，通过 `--test-threads` 修改线程数为 1，可以强制其进行串行测试。

```sh
cargo test -- --test-threads=1
```

Rust 默认会将测试通过的函数内部的 `println!` 打印拦截，仅保留失败信息，通过 `--nocapture` 可以取消拦截。

```sh
cargo test -- --nocapture
```

#### 通过指定测试名称来运行特定测试

```rs
#[cfg(test)]
mod tests {
    #[test] fn case_01() { assert!(true); }
    #[test] fn case_02() { assert!(true); }
    #[test] fn test_01() { assert!(true); }
}

#[cfg(test)]
mod tests2 {
    #[test] fn test_02() { assert!(true); }
}
```

上方示例中存在多个测试模块与测试函数，分析模块树中测试函数的引用路径，如下：

```
tests::case_01
tests::case_02
tests::test_01
test2::test_02
```

`cargo test {path}` 命令会将所有测试函数的路径看做字符串，并使用参数 path 逐一匹配，并执行符合匹配规则的用例。

```sh
cargo test ::c                  # 将执行 tests::case_01 与 tests::case_02
cargo test test                 # 将执行所有用例
cargo test test2::test_02       # 仅执行 test2::test_02
```

#### `--ignored`

通过 `#[ignored]` 注解将某个测试用例标记为 ignored 类别下，执行 `cargo test` 进行自动化测试时，其将被忽略。

之后可以通过 `cargo test -- --ignored` 来单独执行所有被归类至 ignored 类别下的用例。

```rs
#[cfg(test)]
mod tests2 {
    #[test]
    #[ignore]
    fn test_02() { assert!(true); }
}
```
