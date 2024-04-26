# 理解包（package）与单元包（crate）

通过 Cargo 项目工程结构理解包与单元包概念

```sh
.                         // package 包根目录，package 即一个 Cargo 工程，类似 npm 包概念
├── Cargo.toml            // package 声明，包含 package 的名称
└── src
    ├── bin               // bin 目录下每个 rs 文件都会被视为独立的 binary crate 单元包
    ├── lib.rs            // 与 package 同名的 library crate 单元包根节点，也是唯一的 library crate
    └── main.rs           // 与 package 同名的 binary crate 单元包根节点
```

* 一个 package 最多包含一个 library crate，library crate 用于生成库以便于其他项目复用
* 一个 package 可以包含多个 binary crate，每个 binary crate 会生成独立的可执行文件
* `src/main.rs` 被视为与 package 同名的 binary crate 根节点
* `src/lib.rs` 被视为与 package 同名的 library crate，且是唯一的 library crate
* `src/bin` 目录下的每个 rs 文件，均被视为一个单独的 binary crate
* package 内至少需要包含一个 crate（`src/main.rs`、`srclib.rs` 或 `bin/xxx.rs` 必须存在一个）
