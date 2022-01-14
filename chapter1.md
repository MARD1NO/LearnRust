### 安装rust(mac)

```
curl https://sh.rustup.rs -sSf | sh
```

### hello world

新建一个 `hellow_world.rs` 文件

```rust
fn main() {
	println!("Hello, World!"); 
}
```

然后跟cpp一样，先编译再运行

```shell
rustc hellow_world.rs
./hello_world
```

### 使用Cargo

Cargo是Rust工具链内置的构建系统及包管理器

cargo 构建项目

```
cargo new hello_cargo
```

里面的TOML是一种标准的配置格式

构建项目：

```
cargo build
```

然后会在debug目录下找到可执行文件

如果想一次性完成编译和运行，那么就

```
cargo run
```

如果想快速检查当前代码是否可以通过编译，可以

```
cargo check
```

默认是用Debug模式编译

而Debug模式编译速度快，性能不佳（因为没开编译优化选项）。而Release模式性能好，也通常需要更多的时间编译

```
cargo run --release
```

