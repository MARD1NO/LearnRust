# chapter9 错误处理

Rust里将错误分为两类：**可恢复错误**和**不可恢复错误**

### 不可恢复错误与panic！

panic！宏执行时打印出一段错误提示信息，展开并清理当前的调用栈

> 当panic发生时，程序会默认开始栈展开。Rust会沿着调用栈的反向顺序遍历所有调用函数，并依次清理这些函数的数据。除了展开我们还可以选择立即终止程序，由系统os来回收。默认行为panic是展开，如果想要变成终止，则需要在Cargo.toml文件中的[profile]区域添加panic='abort'。

### 使用panic！产生的回溯信息

```rust
RUST_BACKTRACE=1 cargo run
```

注意要获取这些信息的回溯，需要以debug模式运行

### 可恢复错误与Result

大部分的错误没有严重到要让程序停止运行的地步。

以`File::open`为例，其返回类型是`Result<std::fs::File, std::io::Error>`，同时表明这个调用可能成功也可能失败，那么我们就需要对应处理：

```rust
use std::fs::File; 

fn main() {
  let f = File::open("hello.txt"); 
  
  let f = match f {
    Ok(file) => file, 
    Err(error) => {
      panic!("There was a problem opening the file. "); 
    }, 
  }; 
}
```

### 匹配不同的错误

不是所有的错误都是要直接panic的，我们可以再往里继续细分：

```rust
use std::fs::File; 
use std::io::ErrorKind; 

fn main() {
  let f = File::open("hello.txt");
  
  let f = match f {
    Ok(file) => file, 
    Err(error) => match error.kind() {
			ErrorKind::NotFound => match File::create("hello.txt") {
        Ok(fc) => fc, 
        Err(e) => panic!("Error in create file. "), 
      }, 
      other_error => println!("Other error."), 
    }, 
  };
}
```

### 失败时触发panic的快捷方式：unwrap和expect

```rust
use std::fs::File; 

fn main() {
  let f = File::open("hello.txt").unwrap(); 
}
```

当文件打开失败时，就会触发panic

```rust
xxx.expect("Failed xxx")； 
```

expect里面的字符串信息则会包括在panic内

### 传播错误

除了可以在函数内处理错误，我们也可以将错误返回给调用者，让调用者决定如何处理。

```rust 
use std::io; 
use std::io::Read;
use std::fs::File; 

fn read() -> Result<String, io::Error> {
  match f.read_to_string(&mut s) {
    Ok(_) => Ok(s), 
    Err(e) => Err(e), 
  }
}
```

另一个快捷方式是使用？运算符

```rust
fn read() -> Result<String, io::Error> {
  f.read_to_string(&mut s)?; 
  Ok(s)
}
```

使用？运算符的函数必须返回Result，Option或任何实现了std::ops::Try的类型

