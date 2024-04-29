# HashMap

* HashMap 数据存储在**堆**上
* `HashMap<K, V>` 是同质的，所有的键必须为同一类型，所有的值也必须拥有同一类型。

#### 生成 HashMap

```rs
// HashMap 位于 Rust 标准库中
use std::collections::HashMap;

fn main() {
    // 通过元组数组生成 HashMap
    let map = HashMap::from([("Blue", 10), ("Yellow", 30)]);
    // HashMap 默认实现了 Debug trait，可以使用占位符 `{:?}` 进行序列化
    println!("{:?}", map);

    let teams = vec![String::from("Blue"), String::from("Yellow")];
    let initial_scores = vec![10, 50];
    // 通过 collect() 方法生成，需要显式声明 HashMap 类型
    // collect() 方法可以处理多种数据类型，因此需要显式声明 map 的类型为 HashMap
    // 泛型部分可以通过 _ 占位替代，Rust 可以从动态数组中推导出 map 所包含的元素类型
    let map: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
    println!("{:?}", map);
    
    // 通过 HashMap::new() 函数生成 HashMap 实例；因为后续要对其进行值插入，所以需要声明为 mut
    let mut map = HashMap::new();
    map.insert(String::from("Blue"), 10);
    map.insert(String::from("Yellow"), 50);
    println!("{:?}", map);
}
```

#### `insert`

一旦键值对被插入 HashMap，其所有权就会转移给 HashMap；如果只将值的引用插入，不会引发所有权转移，但可能需要声明**生命周期**保证引用有效性。插入键值对时，Key 相同的情况下，**新值覆盖旧值**。

```rs
use std::collections::HashMap;

fn main() {
    let field_name = String::from("Blue");
    let field_value = 30;

    let mut map = HashMap::new();
    // 插入键值对，将导致变量失去所有权
    map.insert(field_name, field_value);
    println!("{:?}", map);

    // 继续使用 field_name 和 field_value 会引发编译异常
    // println!("field_name: {}, field_value: {}", field_name, field_value);
}
```

#### `get`

```rs
use std::collections::HashMap;

fn main() {
  let map = HashMap::from([("Blue", 10), ("Yellow", 30)]);

  // get 函数返回 Option 枚举型，因此需要处理 None 值情况
  let blue = map.get("Blue").expect("None");
  println!("{}", blue);
}
```

#### 更新 HashMap

```rs
use std::collections::HashMap;

fn main() {
    let text = "hello world wonderful world";
    let mut map = HashMap::new();

    // 遍历字符串切片中的单词
    for word in text.split_whitespace() {
        // entry() 方法用于检测 HashMap 中键值对的存在，其返回一个 Entry 枚举，若不存在则通过 or_insert 插入默认值 0
        // or_insert() 方法会返回一个 (&mut V) 的可变引用，因此 count 是一个 `&mut i32` 引用
        let count = map.entry(word).or_insert(0);

        // 为了通过可变引用进行赋值操作，需要使用 * 对 count 解引用
        *count += 1;
    }
}
```