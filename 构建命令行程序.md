通过编写一个简单游戏练习 Rust 基础概念。该程序生成一个 1 到 100 的随机数，玩家输入一个数猜大小。

新建一个项目：

```
$ cargo new guessing_game
$ cd guessing_game
```

## 处理用户输入

允许玩家输入一个猜测的值， *src/main.rs*

```rust
use std::io;

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");
}
```

首先引入 Rust 的标准库 io 输入/输出模块。

```
use std::io;
```

Rust 会默认为所有程序引入一些标准库函数，称为 _prelude_。`std::io` 没有默认引入则需要用关键字 `use` 引入。

使用 `let` 语句创建变量存储用户输入

```
let mut guess = String::new();
```

变量默认时不可变的，在变量名前添加 `mut` 使变量可变。`String::new` 返回一个 `String` 类型的实例。`::` 表示 `new` 是 `String` 类型的关联函数，`new` 函数创建一个空字符串。

从 `io` 模块调用 `stdin` 函数处理用户输入

```
io::stdin() .read_line(&mut guess)
```

`stdin` 函数返回一个 `std::io::Stdin` 实例，代表终端标准输入的句柄。接着使用 `read_line` 方法获取用户输入，`&mut guess` 参数表示使用哪个字符串去存储用户输入。？说明该参数是指定变量引用。引用是 Rust 很重要的特性，它可以在多处访问被指定引用变量的数据，而不需要在内存中重复复制。对引用同样需要在参数名前添加 `&mut` 指定该引用的值可变。

`read_line` 方法返回一个 `Result` 实例，`Result` 是一个枚举类型。`Result` 的可能状态分别是 `Ok` 和 `Err`，`Ok` 代表操作成功且包含生成的值，`Err` 代表失败且包含失败的原因。

`Result` 的实例有一个 `expect` 方法，如果 `Result` 是 `Err`，调用 `expect` 将导致程序崩溃并在终端显示你作为参数传递给 `expect` 的消息。如果 `Result` 是 `Ok`，调用 `expect` 将返回 `Ok` 包含的值。

## 生成随机数

我们正在构建的是一个可执行的 _binary crate_，能被其他程序引入使用但不能自己执行的程序包称为 _library crate_。

我们将使用 `rand` crate 生成随机数，修改  *Cargo.toml* ，在 [dependencies] 部分下添加  `rand` crate 依赖。

```
[dependencies]
rand = "0.8.3"
```

版本号 `0.8.3` 实际上是 `^0.8.3` 的简写，表示任何至少为 `0.8.3` 但低于 `0.9.0` 的版本。

执行项目构建，Cargo 先从注册表中获取依赖项所需的所有最新版本，注册表是来自 Crates.io 的数据副本，Crates.io 是 Rust 生态系统中的人们发布他们的开源 Rust 项目供其他人使用的地方。然后 Cargo 检查 [dependencies] 部分并下载列出的所有尚未下载的 `crates`，本项目中除了会下载 `rand` 还会下载 `rand` 依赖的 `crates`。最后和依赖一起编译该项目。

每次执行构建，Cargo 不会重复下载已下载的依赖。

执行 `cargo update` 命令，Cargo 会忽略 _Cargo.lock_ 并下载所有依赖指定范围内的最新版本和更新 _Cargo.lock_ 指定的版本号。假设 `rand crate` 新发布了版本 `0.8.4` 和 `0.9.0`，执行  `cargo update` 会将 `rand` 更新到 `0.8.4`。

```
$ cargo update
    Updating crates.io index
    Updating rand v0.8.3 -> v0.8.4
```

如果要使用 `rand` 版本 `0.9.0` 或 `0.9.x` 系列中的任何版本，必须将 _Cargo.toml_ 文件更新为如下所示：

```
[dependencies]
rand = "0.9.0"
```

更新 _src/main.rs_ 代码如下：

```rust
use std::io;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    println!("The secret number is: {secret_number}");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");
}
```

添加一行  `use rand::Rng`，这里 `Rng` 指的不是模块是 `trait`，`trait` 定义了随机数生成器需要实现的方法，有点类似于其它语言的 `interface`。

`rand::thread_rng` 函数为我们提供特定的随机数生成器，然后调用 `gen_range` 方法，此方法由我们引入作用域的 `Rng trait` 定义。`gen_range` 方法接收一个范围表达式作为参数，格式是 `start..=end`，包含上下边界。

## 比较用户输入和生成的随机数

```rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    // --snip--

	let mut guess = String::new();

	io::stdin()
		.read_line(&mut guess)
		.expect("Failed to read line");

	let guess: u32 = guess.trim().parse().expect("Please type a number!");

    println!("You guessed: {guess}");

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
}
```

引入一个新的枚举类型 `std::cmp::Ordering`，有三种状态，分别是`Less`, `Greater`, and `Equal`。

使用 `cmp` 方法比较两个值，返回 `Ordering` 其中一种状态。然后使用 `match` 表达式匹配相应的返回值，执行相应的操作。

由 `let mut guess = String::new()` 可以推断出 `guess` 是字符串类型，而 `secret_number` 是数字类型，不能直接比较两个值。需要把 `guess` 转换为数字类型：

```rust
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

Rust 是一个强静态类型系统，我们不可以直接改变 `guess` 的类型，但 Rust 允许我们复用变量名，以此转换数据类型。`trim` 方法可以消除 `guess` 开头和结尾的任何空字符。`parse` 方法可以将字符串转换成任意类型，在变量名后使用冒号 (`:`) 跟类型，标注确切的变量类型。

`parse` 方法返回值是个 `Result`，类似于 `read_line` 的处理，使用 `expect` 方法，如果返回值是 `Err Result`，程序崩溃并把作为参数传递个 `expect` 的消息打印到终端，如果返回值是 `Ok Result`，返回转换后的数字。

## 通过循环猜多次

```rust
	// --snip--
	println!("The secret number is: {secret_number}");

	loop {
		println!("Please input your guess.");

		// --snip--

		match guess.cmp(&secret_number) {
			Ordering::Less => println!("Too small!"),
			Ordering::Greater => println!("Too big!"),
			Ordering::Equal => {
				println!("You win!");
				break;
			}
		}
	}
}
```

将代码放入循环体内，当用户猜中了数字，程序结束。如果玩家输入一个非数字答案，程序就会崩溃退出，

如何处理用户的无效输入

```rust
	// --snip--

	io::stdin()
		.read_line(&mut guess)
		.expect("Failed to read line");

	let guess: u32 = match guess.trim().parse() {
		Ok(num) => num,
		Err(_) => continue,
	};

	println!("You guessed: {guess}");

	// --snip--
```

将 `parse` 方法后的 `expect` 调用转化成 `match` 表达式，如果 `parse` 返回 `Err Result`，匹配到 `Err(_)`，这里下划线是指无论匹配到任何错误，都会执行 `continue`，程序会进行下一轮的迭代。

最终完整的代码：

```rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {guess}");

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}

```
