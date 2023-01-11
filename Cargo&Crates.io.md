## 自定义构建

在 Rust 中，控制版本发布的配置是预先定义好的并且允许开发者自定义。当运行 `cargo build` 时使用 `dev` 配置，运行 `cargo build --release` 时使用 `release` 配置。

Cargo 对 `dev` 和 `release` 有默认配置，在 _Cargo.toml_ 文件中添加 `[profile.*]` 部分可以自定义配置。以下是 `dev` 和 `release` 配置的 `opt-level` 的默认值：

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

`opt-level` 控制编译时的优化程度，取值范围是 0 到 3。相应的优化程度越强，编译时间越长，代码运行效率越高。

你可以覆盖默认值，例如：

```toml
[profile.dev]
opt-level = 1
```

## 发布 Crate 到 Crates.io

_Crates.io_ 是开源分享 Rust 代码的网站，你可以通过发布自己的 package 与其他人共享你的代码。

### 发布代码前做有用的代码注释

准确的注解你的 package 将帮助其他用户知道如何正确有效的使用它们。Rust 有一种特殊的文档注释语法，它可以生成 HTML 文档。文档注释使用三个斜线 `///`，并且支持格式化文本的 Markdown 语法。示例：

````rust
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let arg = 5;
/// let answer = my_crate::add_one(arg);
///
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
````

以上代码的文档注释对 `add_one` 函数的功能进行了介绍，并且提供了如何应用的示例代码。运行 `cargo doc` 可以在 _target/doc_ 目录生成 HTML 文档。运行 `cargo doc --open` 可以接着在浏览器打开生成的文档。

文档注释中的示例代码除了可以演示如何使用你的代码，还可以在运行 `cargo test` 的时候作为测试用例运行。

文档注释语法 `//!` 通常放在 crate 的入口文件（_src/lib.rs_）的开头，注释整个 crate 或模块，用来解释总体用途。例如：

_src/lib.rs_

```rust
//! # My Crate
//!
//! `my_crate` is a collection of utilities to make performing certain
//! calculations more convenient.

/// Adds one to the number given.
// --snip--
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

### 使用 pub use 导出方便使用的公共 API

使用你的 crate 的人不像你那样熟悉结构，如果你的 crate 有一个很复杂的模块层次结构，他们可能很难找到想要使用的部分。

你可以使用 `pub use` 重新导出项目以创建不同于内部结构的公共结构。例如一个 包含两个模块的 crate：

_src/lib.rs_

```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub mod kinds {
    /// The primary colors according to the RYB color model.
    pub enum PrimaryColor {
        Red,
        Yellow,
        Blue,
    }

    /// The secondary colors according to the RYB color model.
    pub enum SecondaryColor {
        Orange,
        Green,
        Purple,
    }
}

pub mod utils {
    use crate::kinds::*;

    /// Combines two primary colors in equal amounts to create
    /// a secondary color.
    pub fn mix(c1: PrimaryColor, c2: PrimaryColor) -> SecondaryColor {
        // --snip--
        unimplemented!();
    }
}
```

依赖于这个库的其他 crate 如果想要使用 `PrimaryColor` 和 `mix` 需要做如下引入：

```rust
use art::kinds::PrimaryColor;
use art::utils::mix;

fn main() {
    let red = PrimaryColor::Red;
    let yellow = PrimaryColor::Yellow;
    mix(red, yellow);
}
```

其他开发者使用前必须弄清楚 `PrimaryColor` 在 `kinds` 模块中，而 `mix` 在 `utils` 模块中。为避免其他人需要了解代码的内部接口，可以使用 `pub use` 将要使用的项目重新导出到顶层目录。

```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub use self::kinds::PrimaryColor;
pub use self::kinds::SecondaryColor;
pub use self::utils::mix;

pub mod kinds {
    // --snip--
}

pub mod utils {
    // --snip--
}
```

现在可以更方便的导入 `PrimaryColor` 和 `mix`：

```rust
use art::mix;
use art::PrimaryColor;

fn main() {
    // --snip--
}
```

### 设置 Crates.io 帐户

使用 GitHub 账户登录  [crates.io](https://crates.io/) ，获取 API key，在终端运行 `cargo login`：

```shell
$ cargo login abcdefghijklmnopqrstuvwxyz012345
```

### 添加元数据到将要发布的 crate

在发布之前，您需要在 crate 的 _Cargo.toml_ 文件的 `[package]` 部分添加一些元数据。

首先需要一个唯一的名称，crates.io 上的 crate 名称按照先到先得的原则分配。然后描述和 `license` 也是必须的。可以指定多个由 `OR` 分隔的许可证标识符。

_Cargo.toml_

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

### 发布到 Crates.io

因为发布是永久性的，所以版本不可覆盖，代码不可删除。使用 `cargo publish` 发布：

```shell
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished dev [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
```

### 发布新版本

更改 Cargo.toml 文件中指定的 `version` 值并重新发布。

### 弃用某个版本

虽然你不能删除以前版本的 crate，但你可以阻止任何未来的项目将它们添加为新的依赖项，同时允许所有依赖它的现有项目可以继续运行。

运行 `cargo yank` 并指定要弃用的版本。

```shell
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

通过将 `--undo` 添加到命令，你还可以撤消 `yank` ：

```shell
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

## Cargo 工作区

Cargo 提供了一个称为工作区的功能，可以协同管理多个功能相关的 packages。工作区内的 packages 共享 _Cargo.lock_ 和编译后的输出目录。

首先新建一个项目根目录：

```shell
$ mkdir add
$ cd add
```

然后新建一个 _Cargo.toml_ 文件，该文件以 `[workspace]` 开头，添加所有 packages 的路径到 members，如下：

_Cargo.toml_

```toml
[workspace]

members = [
    "adder",
]
```

在当前目录运行 `cargo new` 添加 `adder` 二进制 crate。

```shell
$ cargo new adder
     Created binary (application) `adder` package
```

一个工作区只有一个 _target_ 目录，存放所有 crates 编译后的文件。可以避免每个 crate 的重复编译。

添加另一个 package add_one：

_Cargo.toml_

```toml
[workspace]

members = [
    "adder",
    "add_one",
]
```

```shell
$ cargo new add_one --lib
     Created library `add_one` package
```

在  *add_one/src/lib.rs* 文件添加函数 add_one：

_add_one/src/lib.rs_

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

在 adder 添加 add_one 的依赖：

_adder/Cargo.toml_

```toml
[dependencies]
add_one = { path = "../add_one" }
```

现在可以在 adder 内使用 add_one 函数了，如下：

_adder/src/main.rs_

```rust
use add_one;

fn main() {
    let num = 10;
    println!("Hello, world! {num} plus one is {}!", add_one::add_one(num));
}
```

在 add 目录执行编译：

```shell
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.68s
```

从 add 目录运行二进制 crate，我们可以使用 -p 参数和 package 名来指定我们要在工作区中运行的 package：

```shell
$ cargo run -p adder
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

### 工作区中的外部依赖

工作区内只有一个 _Cargo.lock_，所以所有的 crates 都使用相同版本的依赖，这可以确保工作区内的 crates 相互兼容。而且即使多个 crates 引入同一个外部依赖，该依赖也只会被下载一次，节省了工作区的空间。

### 工作区添加测试功能

给 `add_one` 添加测试：

_add_one/src/lib.rs_

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(3, add_one(2));
    }
}
```

在 add 目录运行 `cargo test` 会执行所有 crates 的测试。使用 `-p`  参数可以指定要运行测试的 crate。

```shell
$ cargo test -p add_one
    Finished test [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-b3235fea9a156f74)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

## cargo install 安装二进制 crate

`cargo install` 可以安装其他人在 crates.io 上共享的工具。只能安装二进制 crate，二进制 crate 是指可运行的程序，即 crate 有一个 src/main.rs 文件或另一个被指定的文件。

使用 cargo install 安装的所有二进制文件都存储在安装根目录的 bin 文件夹中。这个目录一般是 _$HOME/.cargo/bin_。
