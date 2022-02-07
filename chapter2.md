# chapter2 编写一个猜数游戏



### 处理一次预测

```rust
use std::io;

fn main() {
    println!("Guess the number!"); 
    println!("Please input your guess. "); 

    let mut guess = String::new(); 
    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");
    
    println!("You guessed: {}", guess); 
}
```

- `use std::io` 将标准库的io模块引入当前作用域中
- `let mut guess = String::new();` 创建一个存储输入数据的变量，其中mut关键字代表这个变量可变

然后调用io模块的stdin的`read_line`方法来获取用户的输入，其中参数列表里表示输入内容要存储到哪个变量，这里我们存储到了可变的`guess`变量。

### 使用Result类型来处理可能失败的情况

readline返回的是`io::Result`值，在Rust标准库中，Result是一个枚举类型，它拥有Ok和Err两个变体。其中expect方法也有不同的行为，如果是Err，则expect方法会中断当前的程序，并将传入的字符串参数显示出来。



即使我们没有在语句末尾调用expect，程序可以编译通过，但是会有相关警告。

### 生成一个保密数字

#### 借助包来获得更多功能

我们将使用rand包来随机生成一个数字，cargo主要功能就是帮助我们管理和使用第三方库

我们给cargo的dependencies区域下方添加依赖：

```shell
[dependencies]

rand = "0.3.14"
```

然后调用`cargo build`重新编译

> 如果出现了 Waiting for File Lock on Package Cache Lock 使用下面命令即可：
>
> rm ~/.cargo/.package-cache

#### 通过Cargo.lock文件确保我们的构建是可重现的

第一次构建项目时，Cargo会依次遍历我们声明的依赖及其版本。再次构建项目时，Cargo就会优先检索Cargo.lock，假如文件中存在已经指明具体版本的依赖库，那么它就回跳过计算版本号的过程，直接使用文件中的版本

##### 升级依赖包

它会强制Cargo忽略Cargo.lock文件，并重新计算出所有依赖包中符合Cargo.toml声明的最新版本

基于语义化版本规则，Cargo在自动升级时只会寻找大于0.3.0并小于0.4.0的最新版本

### 生成随机数

```rust
use rand::Rng; 

let secret_number = rand::thread_rng().gen_range(1, 101); 
```

> 你可以使用 cargo doc --open 来生成你当前所有依赖的文档

#### 比较猜测数字

```rust
use std::cmp::Ordering

match guess.cmp(&secret_number){
        Ordering::Less => println!("Too small!"), 
        Ordering::Greater => println!("Too Large!"), 
        Ordering::Equal => println!("You win!"), 
    }
```

Rust有一个静态强类型系统，同时也有自动类型推导能力。前面我们的guess是string类型，我们需要转换成数值类型，才能进行比较：

```rust
let guess: u32 = guess.trim().parse()   
                        .expect("Please type a number"); 
```

Rust允许使用同名的新变量来隐藏旧变量

其中trim返回一个去掉首尾所哟欧空白字符的新字符串，parse方法则尝试将当前的字符串解析为某种数值



我们使用loop和break来完成整个循环和退出

#### 处理非法输入

```rust
let guess: u32 = match guess.trim().parse() {
            Ok(num) => num, 
            Err(_) => continue, 
        }; 
```

我们使用match来处理非法输入，如果是非法输入则重新进入循环

