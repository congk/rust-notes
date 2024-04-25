# trait 特征

Rust 中的 `trait` 定义类似于常规的 `Interface` 定义，但允许其定义中具备默认实现；

具备默认实现的 `trait` 类似于 Typescript 中的**类装饰器**，通过对类型追加注解使类实例的行为发生变化，或符合某种特征要求；

`trait` 用来描述某些可以被特定类型拥有且能够被其他类型共享的功能，使我们能够**以抽象的方式来定义共享行为**；

Rust 中**不支持类继承**的方式来达成代码共享的目的。

```rs
// 定义 Named trait
trait Named {
    // 定义 name 方法但不提供默认实现，类似常规 interface 中的方法定义
    fn name(&self) -> String;

    // 定义 print_name 方法，并提供具体实现
    fn print_name(&self) {
        // 这里调用了 name() 方法，是允许的
        println!("My name is {}!", self.name());
    }
}

struct Cat {
    age: u8,
}
// 为 Cat 结构体实现 Named trait，需要使用 for 关键字
impl Named for Cat {
    // 实现 trait 要求的方法
    fn name(&self) -> String {
        String::from("Cat")
    }
}

struct Dog;
// 为 Dog 结构体实现 Named trait
impl Named for Dog {
    // 实现 trait 要求的方法
    fn name(&self) -> String {
        String::from("Dog")
    }
}

fn main() {
    let cat = Cat { age: 5 };
    cat.print_name();

    let dog = Dog;
    dog.print_name();
}
```

#### trait 的使用限制

* 只有当类型或 trait 其中一个由内部定义时，才能为该类型实现对应的 trait，这是出于**程序一致性原则**考虑。我们不能为**外部实现类型**实现**外部 trait**，以确保其他人编写的内容不会破坏你的代码，反之亦然。
* 无法在重载 trait 方法实现过程中，调用 trait 的默认实现。

```rs
impl Named for Dog {
    // 实现 trait 要求的方法
    fn name(&self) -> String {
        // 在重载 name 方法过程中调用 print_name 方法是非法的，因为 print_name 依赖 name 方法实现。
        // self.print_name();
        String::from("Dog")
    }

    // 重载默认的 print_name 实现
    fn print_name(&self) {
      // 在重载 print_anme 的过程中调用 self.print_name() 也是非法的，Rust 不支持类似 Typescript 中的 super.print_name() 的实现方式
      self.print_name()
    }
}
```

#### 使用 trait 作为参数

```rs
// 通过泛型定义，约束入参类型必须实现 Named trait，以下两种实现方式是等效的
fn get_name<T: Named>(animal: T) -> String { animal.name() }
fn get_name(animal: impl Named) -> String { animal.name() }

// 使用 `+` 语法指定多个 trait 约束
fn sleep<T: Named + Display>(animal: T) {}
fn sleep(animal: impl Named + Display) {}
```

#### 返回实现了 trait 的类型

```rs
fn return_trait() -> impl Display + Clone {
  1
}
```

#### 通过 where 简化 trait 约束

```rs
fn some_func<T: Display + Clone, U: Debug + Clone>(t: T, u: U) -> impl Display + Clone {
  t
}

// 简化为
fn some_func<T, U>(t: T, u: U) -> T
where
    T: Display + Clone,
    U: Debug + Clone,
{
  t
}
```

#### 使用 trait 有条件的实现方法

```rs
struct House<T> {
  owner: T
}

// 指定仅对实现了 Display + Clone trait 的泛型增加实现
impl<T: Dog + Cat> House<T> {
  fn get_owner_name(&self) -> String {
    self.owner.name()
  }
}
```

#### 动态派发

Rust 编译器会在泛型使用 trait 约束时，执行**单态化**：编译器会为每一个具体类型生成对应泛型类型和泛型方法的非泛型实现，并使用这些具体的类型来替换泛型参数。通过单态化生成的代码会执行**静态派发（static dispatch）**，这意味着编译器能够在编译过程中确定你调用的具体放。这个概念与**动态派发（dynamic dispatch）**相对应，动态派发下的编译器无法在编译阶段确定你要调用的究竟是哪个方法。在进行动态派发的场景中，编译器会生成一些额外的代码以便在运行时找出我们希望调用的方法。

```rs
// 定义一个 Draw trait
pub trait Draw {
    fn draw(&self);
}

pub struct Screen {
    // components 是一个实现了 Draw trait 的实例数组
    pub components: Vec<Box<dyn Draw>>,
}
impl Screen {
    pub fn run(&self) {
        // 遍历 components 实例
        for component in self.components.iter() {
            component.draw()
        }
    }
}

// 定义 Button 并实现 Draw
pub struct Button {}
impl Draw for Button {
    fn draw(&self) {
        println!("Draw button")
    }
}

// 定义 SelectBox 并实现 Draw
pub struct SelectBox {}
impl Draw for SelectBox {
    fn draw(&self) {
        println!("Draw select-box")
    }
}

fn main() {
    let screen = Screen {
        components: vec![Box::new(SelectBox {}), Box::new(Button {})],
    };
    screen.run();
}
```

Rust 必然会在我们使用 trait 对象时执行动态派发。因为编译器无法知晓所有能够用于 trait 对象的具体类型，所以它无法在编译时确定需要调用哪个类型的哪个具体方法。不过 Rust 会在运行时通过 trait 对象内部的指针去定位具体调用哪个方法，该定位过程会产生一些不可避免的运行时开销，而这并不会出现在静态派发中。动态派发还会阻止编译内联代码，进而使得部分优化操作无法进行，但不管怎样，动态派发确实能够带来额外的灵活性。

#### trait 对象必须保证对象安全

Rust 中只能将一个满足**对象安全（object-safe）**的 trait 转换为 trait 对象（如 `dyn Draw`）。

如果一个 trait 中定义的所有方法满足下面两条规则，那么这个 trait 就是对象安全的：

* 方法的返回类型不是 `Self`
* 方法中不包含任何泛型参数

标准库中的 `Clone trait` 就是一个不符合对象安全的例子，因此无法将其转为 trait 对象。

```rs
pub trait Clone {
  fn clone(&self) -> Self;
}
```

其实现中返回了 `Self`，指代实现了 Clone trait 的具体类型。

```rs
let v: Vec<Box<dyn Clone>> = vec![];    // 编译异常：the trait `Clone` cannot be made into an object
```