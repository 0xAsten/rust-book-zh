## 安装 Rust

将使用命令行工具 `rustup` 下载安装和管理 Rust 版本。

### Linux 或 macOS

下载安装 `rustup` 并同时安装最新版本的 Rust

```
$ curl --proto '=https' --tlsv1.3 https://sh.rustup.rs -sSf | sh
```

安装 C 编译器，一些常见的 Rust 包依赖于 C。并缺 C 编译器包含 _linker_ 程序，可以讲 Rust 编译后的输出合并到一个文件。

```
$ xcode-select --install
```

检查是否安装成功

```
$ rustc --version
```

更新 Rust 到最新版本

```
$ rustup update
```

卸载 Rust

```
$ rustup self uninstall
```

## Hello, World!

新建项目目录 hello_world

```
$ mkdir hello_world
$ cd hello_world
```

新建文件 _main.rs_，输入以下代码并保存

```
fn main() {
    println!("Hello, world!");
}
```

编译和运行

```
> rustc main.rs
> .\main.exe
Hello, world!
```

`main` 通常是一个可执行 Rust 程序的入口函数。

`println!` 是 Rust 的宏，宏是一类特殊的函数，使用 `!` 意味着是宏函数不是普通函数。

## Cargo

Cargo 是 Rust 的构建系统和包管理工具，它可以编译构建代码，下载依赖的包等等。

安装 Rust 时同时安装了 Cargo，验证 Cargo 是否安装

```
$ cargo --version
```

### 使用 Cargo 管理项目

新建项目，观察和不使用 Cargo 的区别

```
$ cargo new hello_cargo
$ cd hello_cargo
```

在 _hello_cargo_ 文件夹目录下生成了 _Cargo.toml_ 配置文件， *src* 目录和 _main.rs_ 文件。

使用 Cargo 可以帮助你组织项目结构。

在 _hello_cargo_ 目录下，使用 Cargo 构建项目

```
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

该命令创建一个可执行的二进制文件存放在 _target/debug/hello_cargo_，可以直接运行

```
$ ./target/debug/hello_cargo # or .\target\debug\hello_cargo.exe on Windows
Hello, world!
```

第一次运行 `cargo build`，还会新建一个文件  *Cargo.lock*，该文件跟踪项目依赖 _Packages_ 的准确版本号，在 Rust，packages 被称为 _crates_。永远不用手动修改该文件，Cargo 会帮你管理。

可以使用一条 `cargo run` 命令编译并执行

```
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

使用 `cargo check` 快速检查代码可编译但不会生成可执行文件，有利于开发过程中快速检验代码。

```
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

使用  `cargo build --release` 优化编译，可执行文件最终存储在  *target/release*，该命令需要更长的编译时间但执行效率更高，通常用于构建最终需要发布的程序。
