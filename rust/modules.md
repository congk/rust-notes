# 模块

模块允许将 crate 内的代码按照可读性、易用性进行分组，并控制条目的私有性（或封装性）。

```rs
// 通过 mod 关键字定义模块
mod foo {
    // Rust 中默认所有条目（结构体、枚举、函数、模块、方法、常量）都是私有的，通过 pub 关键字开放访问。
    // 父模块无法访问子模块内的私有条目，子模块可以获取到所有父模块的所有条目
    pub fn bar() {}

    // 对结构体声明为 pub，不影响其内部字段和方法的私有性
    pub struct User {
        pub name: String,
        age: u8,
    }

    // 将枚举类型定义为 pub，那么其所有变体均为 pub
    pub enum IpAddr {
        V4,
        V6,
    }
}

fn main() {
    // 使用 crate 进行绝对路径调用，crate 为模块所属单元包，是模块树的根节点
    crate::foo::bar();

    // 相对路径调用，因 test 函数与 foo 模块是同级条目
    foo::bar();
}

mod test {
    fn call() {
        // 通过 super 父级模块相对路径调用，super 指向 foo2 模块
        // super::super 是非法的，超出两级的情况下只能使用绝对路径，或使用 use 将条目导入作用域
        super::foo::bar();
    }

    mod test2 {
        // 使用 use 将条目导入当前作用域，use 可以用于除结构体方法外的所有 pub 条目导入
        // use 使用相对路径时，需要在路径开始出增加 self 关键字，如 `use self::super::xxx` 或 `use self::xxx`
        use crate::foo::bar;
        fn testFn() {
            // 直接调用条目
            bar();
        }

        // use 引入条目时，可以通过 as 关键字对其重新命名。
        use crate::foo::User as SomeOne;

        // use 引入作用域的条目默认作为作用域内的私有条目，可以搭配 pub 关键字对外重新导出
        pub use crete::foo::IpAddr;

        // use 语句可以嵌套使用以导入同一路径下的不同条目
        use crate::foo::{ User, IpAddr, bar };

        // 或使用通配符 * 导入路径中的所有公共条目
        use crate::foo::*;
    }
}
```

## 多文件模块组织

```rs
// src/foo.rs

// 定义 foo 模块
pub mod foo {
}
```

```rs
// src/lib.rs

// 使用 mod 声明模块，Rust 会前往<<同级目录的 foo.rs 或 foo/mod.rs >>加载模块内容。
mod foo;

fn main() {}
```