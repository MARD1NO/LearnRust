# Chapter6 枚举与模式匹配

### 定义枚举

```rust
enum IpAddrKind{
  V4, 
  V6, 
}
```

枚举允许我们将其关联的数据嵌入到枚举变体内，比如这段代码：

```rust
let home = IpAddr{
    kind: IpAddrKind::V4, 
    address: String::from("127.0.0.1"), 
}; 
```

可以简化为：

```rust
enum IpAddr {
    V4(String), 
    V6(String), 
}

fn main() {
    let home = IpAddr::V4(String::from("127.0.0.1"));
}
```

每个变体可以拥有不同类型和数量的关联数据：

```rust
enum IpAddr {
    V4(i32, i32, i32), 
    V6(String), 
}
```

我们看另外一个枚举：

```rust
enum Message{
  Quit, 
  Move {x: i32, y: i32}, 
  Write(String), 
  ChangeColor(i32, i32, i32), 
}
```

- Quit没有任何关联数据
- Move 包含了一个匿名结构体
- Write包含了一个String
- ChangeColor 包含了3个i32值

当然我们可以分别实现出四个不同的结构体，但具体差别是，每个结构体拥有自己的类型，而`Message`枚举不同，它是一个统一的类型

### Option枚举及其在空值处理方面的优势

Option是标准库的一个枚举，在Rust里没有空值这个概念，因为空/非空的属性被广泛散布在程序中，很容易引发问题



Rust用Option枚举来标识一个值无效：

```rust
enum Option<T> {
  Some(T), 
  None, 
}
```

当我们使用None的时候，需要明确指出T的具体类型（不然编译器推断不出来）

```rust
let some_number = Some(5); 
let absent_number: Option<i32> = None; 
```

> 为什么Option就比Null好？
>
> Option<T>和<T>属于不同类型，编译器不允许我们直接相加。你需要把Option<T>转换成T。而此时我们猜需要考虑，显式地处理为空的情况。而如果是T，那这个值就一定非空

### 控制流运算符match

```rust
fn value_in_cents(coin: Coin) -> u32{
  match coin {
    Coin::Penny => 1, 
    Coint::Nickel => 2, 
  }
}
```

match是将一个值与一系列模式进行比较，比如第一个采取了`Coin::Penny`作为模式，并用`=>`将模式和代码隔开。

### 绑定值的模式

匹配分支可以绑定被匹配对象的部分值：

```rust
enum UsState{
  Alabama, 
  Alaska, 
}

enum Coin{
  Penny, 
  Quarter(UsState)
}

fn value_in_cents(coin: Coin) -> u32{
  match coin {
    Coin::Penny => 1, 
    Coint::Quarter(state) => {
      println!("State quarter from {:?}!", state); 
      return 25; 
    }, 
  }
}
```

### 匹配Option

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
  match x {
    None => None, 
    Some(i) => Some(i+1), 
  }
}
```

### 匹配必须穷举所有的可能

Rust的match必须穷尽所有可能性，来保证代码合法

### _ 通配符

匹配其他情况可以用这个通配符：

```rust
let u8 = 0u8; 
match u8 {
  1 => println!("one"), 
  2 => println!("two"), 
  _ => println!("Any"), 
}
```

### 简单控制流 if let

```rust
if let Some(3) = some_u8_val {
  println!("Three"); 
} else {
  println!("Not Three");
}
```



