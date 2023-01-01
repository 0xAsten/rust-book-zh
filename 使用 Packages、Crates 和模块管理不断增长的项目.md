## Packages 和 Crates

crate 是 Rust 编译器一次考虑的最小代码量。crate 有两种形式，binary crate 和 library crate。

_Binary crates_ 是可以编译为可执行文件的程序，每个 crate 必须有一个 main 函数。

_Library crates_ 没有 main 函数且不可以编译为可执行文件，它们旨在定义多个项目共享的功能。

_package_ 是提供一组功能的一个或多个 crates 的捆绑包。一个 package 包含一个 Cargo.toml 文件，该文件描述了如何构建这些 crates。

一个 package 可以包含任意多个 binary crates，但最多只能包含一个 library crate。一个 package 必须至少包含一个 crate，无论是 binary crate 还是 library crate。

## 定义模块以控制作用域和可见性

### 模块编译规则

- 编译 crate 时，编译器首先查看 crate 根文件（通常 src/lib.rs library crate 或 src/main.rs 用于 binary crate）。

- 在 crate 根文件可以声明新的模块，假设使用 mod garden; 命令声明一个 “garden” 模块，编译器会在以下位置找模块的实现代码：

  - 跟在 mod garden 声明后的大括号内
  - 在文件 src/garden.rs 中
  - 在文件 src/garden/mod.rs 中

- 在 crate root 以外的任何文件中，都可以声明子模块。例如，可以在 src/garden.rs 中声明 mod vegetables;。编译器将在以父模块命名的目录中的这些地方查找子模块的代码：

  - 跟在 mod vegetables 声明后的大括号内
  - 在文件 src/garden/vegetables.rs 中
  - 在文件 src/garden/vegetables/mod.rs

- 一旦一个模块成为你的 crate 的一部分，你就可以使用代码的路径从同一个 crate 中的任何其他地方引用该模块中的代码，只要隐私规则允许。例如，garden/vegetables 模块中的 Asparagus 类可以在 crate::garden::vegetables::Asparagus 中找到

- 默认情况下，模块中的代码对其父模块是私有的不可见的。要使模块公开，请使用 pub mod 而不是 mod 来声明它。要使公开模块中的元素也公开，请在它们的声明之前使用 pub。

- 在作用域内，use 关键字减少长路径的重复书写。在任何可以引用 crate::garden::vegetables::Asparagus 的作用域内，您可以使用 use crate::garden::vegetables::Asparagus 创建一个快捷方式；从那时起，您只需编写 Asparagus 即可在作用域内使用该类型。

模块让我们可以将代码组织在一个 crate 中，以提高可读性和重用性。模块还允许我们控制项目的可见性。

src/main.rs 和 src/lib.rs 被称为 crate 的 root，因为这两个文件在 crate 模块结构的根部形成一个名为 crate 的模块，这就是模块树。

模块树就像文件系统中的目录一样，您可以使用模块来组织代码。就像目录中的文件一样。

## 引用路径

我们使用路径的方式引用模块树中的元素。

路径的两种形式：

- 绝对路径是从 crate root 开始的完整路径；对于来自外部 crate 的代码，绝对路径以该 crate 的名称开头，对于来自当前 crate 的代码，它以文字 crate 开头。

- 相对路径从当前模块开始，使用 self、super 或当前模块中的标识符。

路径引用用双冒号 (::) 分隔。示例：

src/lib.rs

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

eat_at_restaurant 函数是我们 library crate 的公共 API 的一部分，我们用 pub 关键字标记它，它分别用绝对路径和相对路径调用了 add_to_waitlist 方法。

第一次调用 add_to_waitlist 函数使用了绝对路径，用 crate 从 crate root 开始就像在 shell 中使用 / 从文件系统 root 开始一样。

第二次用相对路径，路径从 front_of_house 开始，front_of_house 和 eat_at_restaurant 在模块树中处于相同层级。

在 Rust 中，所有元素（函数、方法、结构、枚举、模块和常量）默认情况下对父模块都是不可见的私有的。使用 pub 关键字可以将子模块代码的内部部分公开给外部祖先模块。

我们希望父模块中的 eat_at_restaurant 函数能够访问子模块中的 add_to_waitlist 函数，所以需要用 pub 关键字标记 hosting 模块。公开 hosting 模块并不会公开其中的内容，所以还需要用 pub 标记 add_to_waitlist 函数。

因为 eat_at_restaurant 和 front_of_house 是同辈关系，所以不需要标记 pub。

### 使用 super 相对路径

我们可以通过在路径的开头使用 super 来构建从父模块开始的相对路径，而不是当前模块或 crate root。这就像使用 .. 语法开始一个文件系统路径。示例：

src/lib.rs

```rust
fn deliver_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::deliver_order();
    }

    fn cook_order() {}
}
```

fix_incorrect_order 函数在 back_of_house 模块中，因此我们可以使用 super 转到 back_of_house 的父模块，在本例中是 crate root。

### 使用 pub 标记 struct 和枚举类型

如果我们在 struct 定义之前使用 pub，我们将 struct 公开，但 struct 的字段仍然是私有的。我们可以根据具体情况公开或不公开每个字段。

```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
}

pub fn eat_at_restaurant() {
    // Order a breakfast in the summer with Rye toast
    let mut meal = back_of_house::Breakfast::summer("Rye");
    // Change our mind about what bread we'd like
    meal.toast = String::from("Wheat");
    println!("I'd like {} toast please", meal.toast);

    // The next line won't compile if we uncomment it; we're not allowed
    // to see or modify the seasonal fruit that comes with the meal
    // meal.seasonal_fruit = String::from("blueberries");
}
```

因为 back_of_house::Breakfast 结构中的 toast 字段是公共的，所以在 eat_at_restaurant 中我们可以使用点符号写入和读取 toast 字段。请注意，我们不能在 eat_at_restaurant 中使用 seasonal_fruit 字段，因为 seasonal_fruit 是私有的。

相反，如果我们公开一个枚举类型，那么它的所有变体都是公开的。我们只需要 enum 关键字前的 pub。

```rust
mod back_of_house {
    pub enum Appetizer {
        Soup,
        Salad,
    }
}

pub fn eat_at_restaurant() {
    let order1 = back_of_house::Appetizer::Soup;
    let order2 = back_of_house::Appetizer::Salad;
}
```

## 使用 use 关键字将路径引入作用域

我们可以使用 use 关键字创建路径的快捷方式，然后在作用域内的其他地方使用较短的名称。

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

use 只会为当前作用域创建快捷方式，如果将 eat_at_restaurant 函数移动到名为 customer 的新子模块中，则不能通过编译，如下：

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

mod customer {
    pub fn eat_at_restaurant() {
        hosting::add_to_waitlist();
    }
}
```

使用 use 将函数带入作用域的惯用方式是用 use 将函数的父模块带入作用域，意味着我们必须在调用函数时指定父模块。

另一方面，当使用 use 引入 struct、枚举时，习惯上指定完整路径。例如：

```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, 2);
}
```

### 使用 as 关键字提供新名称

对于使用 use 将两种相同名称的类型带入相同作用域的问题，我们可以用 as 指定该类型的别名。例如：

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
    Ok(())
}

fn function2() -> IoResult<()> {
    // --snip--
    Ok(())
}
```

### Re-exporting

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

使用 pub use 将 hosting 引入作用域，同时外部代码可以使用 restaurant::hosting::add_to_waitlist() 调用函数而不需要使用 restaurant::front_of_house::hosting::add_to_waitlist()。

### 使用外部 packages

将第三方 packages 列在你的 Cargo.toml 文件中，并使用 use 将它们的 crate 中的元素引入作用域。

### 使用嵌套路径

```rust
// --snip--
use std::cmp::Ordering;
use std::io;
// --snip--
```

可以把以上两行缩写为一行，指定路径的公共部分，后跟两个冒号，然后用大括号括起路径不同部分的列表，如：

```rust
// --snip--
use std::{cmp::Ordering, io};
// --snip--
```

### 全局引入

```rust
use std::collections::*;
```

此 use 语句将 std::collections 中定义的所有公共元素带入当前范围。
