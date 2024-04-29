# 字符串

* Rust 中字符串使用 `UTF-8` 编码；
* Rust 语言核心部分仅有一种字符串类型 `&str`，即**字符串切片 str**，通常以借用的形式出现；
* `String` 类型是被定义在 Rust 标准库中，是一个基于 `Vec<u8>` 的封装类型；字符串长度 `len()` 实际返回的是 `Vec<u8>` 数组长度。

## 字符串拼接

#### 使用 `+` 运算符拼接字符串

使用 `+` 运算符进行字符串拼接，会调用一个 `fn add(self, s: &str) -> String` 方法，其会夺取第一个入参变量的所有权。

```rs
let s1 = String::from("hello")
let s2 = String::from("world");

// 字符串拼接会导致 s1 丢失所有权，所有权通过 add 方法返回并绑定 s3，继续使用 s1 会引发异常
// &s2 是 &String 类型，Rust 会自动强制类型转换为 &str 切片
let s3 = s1 + ", " + &s2;
```

#### 使用 `format!` 宏拼接字符串

```rs
let s1 = String::from("hello");
let s2 = String::from("world");

// `format!` 不会夺取任何参数所有权
let s3 = format!("{}, {}", s1, s2);
```

#### `push` 与 `push_str`

```rs
let str = String::from("hello");

// 追加 char 字符
str.push(',');

// 追加字符串切片
str.push_str(" world");
```

## 字符串索引

**Rust 字符串不支持索引**，原因在于：

* Rust 中 `char` 型数据容量是 4 Bytes，通过索引并不能总是获取到有效的 Unicode 标量值。
* 性能考虑：Rust 编译器会检查索引是否越界，这意味着使用索引时，Rust 必须遍历所有字符串才能确定究竟有多少个合法字符存在。。

特别注意，**对 String 类型切片也是按字节索引切片，因此也无法保证总能获取到有效的 Unicode 标量值，容易触发 panic 导致程序崩溃**。

```rs
// 汉字 Unicode 标量占 3 个字节
let str = String::from("你好");

let s = &str[0..2];   // 切片取前两个字节，因无法获取到有效的 Unicode 标量值，所以会引发 panic 导致程序崩溃
```

## 遍历字符串：`chars` 与 `bytes`

```rs
// 对每个 Unicode 标量值进行处理，最好使用 chars()
println!("{:?}", "你好".chars());   // output: Chars(['你', '好'])

// 遍历 Unicode 标量值
for c in "你好".chars() {
  println!("{}", c);
}

// "你好" 是两个 char 字符，但占用了 6 个字节
println!("{:?}", "你好".bytes());   // output: Bytes(Copied { it: Iter([228, 189, 160, 229, 165, 189]) })

// 遍历原始字节数据
for b in "你好".bytes() {
  println!("{}", b);
}

// 字符串切片长度按 bytes 进行计算
println!("{}", "你好".len());       // output: 6
```