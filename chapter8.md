# chapter8 通用集合类型

### 使用动态数组

```rust
let v: Vec<i32> = Vec::new(); 
let v = vec![1, 2, 3]; 

// 插入元素
let mut v = Vec::new();
v.push(5); 
v.push(6);
```

### 销毁动态数组也会销毁其中的元素

### 读取动态数组中的元素

```rust
// 1. 索引，如果越界则会触发panic
let v = vec![1, 2, 3]; 
let third: &i32 = &v[2]; 

// 2. get，返回的是一个Option<&T>
match v.get(2) {
  Some(third) => println!("The third element is: {}", third), 
  None => println!("There is no third element. "), 
}
```

下面的代码编译会出现错误：

```rust
let mut v = vec![1, 2, 3, 4, 5]; 
let first = &v[0]; 

v.push(6);
```

此时已经持有了动态数组的一个不可变引用，而往动态数组添加元素的时候，可能出现空间不够，需要分配新的内存空间。那不可变引用指向的内存可能就被释放。

### 遍历动态数组中的值

```rust
for i in &v {
  // ...
}

for i in &mut v{
  *i += 50; 
}
```

### 使用枚举来存储多个类型的值

动态数组只能存储相同类型的值，此时我们可以使用枚举(因为枚举中的所有变体都被定义为同一种枚举类型)

```rust
enum Cell{
  Int(i32), 
  Float(f64), 
  Text(String), 
}

let row = vec![Cell::Int(3), Cell::Float(5.0), Cell::Text(String::from("blue"))]; 
```

### 使用字符串

Rust在语言核心部分只有字符串切片`str`，`String`类型被定义在Rust标准库中。

```rust
let mut s = String::new(); 

// 如果类型实现了 Display trait，则可以：
let data = "xxx"; 
let s = data.to_string(); 

// 用String::from
let s = String::from("xxx"); 
```

### 更新字符串

`push_str`向字符串添加字符串

`push`向字符串添加字符

### 使用+运算符或format！宏来拼接字符串

```rust
let s1 = String::from("xxx"); 
let s2 = String::from("xxx"); 
let s3 = s1 + &s2; // 这里s1已经被move移动，再也不能被使用
```

其实调用的是add方法：

```rust
fn add(self, s: &str) -> String {...}
```

它获取了第一个字符串的所有权，用第二个字符串的引用与其相加

可以看到第二个参数的类型是`&str`，与我们传进来的`&String`并不相同。Rust使用了一种解引用强制转换的技术，将`&s2`转换为`&s2[..]`

### 字符串索引

字符串不支持直接索引，String实际上是一个基于Vec<u8>的封装类型，对于不同的语言，存储所用的字节数不一样

更具体来说，字符串中的数据可以被看作成字节，标量值，字形簇(这个最接近人们眼中字母的概念)

### 字符串切片

```rust
let hello = String::from("Hello"); 
let s = &hello[0..2];
```

[]里面填写范围来**指定所需的字节内容**

### 遍历字符串的方法

```rust
// 1. 以字符形式遍历
for c in s.chars() {
  println!("{}", c);
}

// 2. 以字节标量值遍历
for c in s.bytes() {
  println!("{}", c);
}
```

### 在哈希映射中存储键值对

```rust
use std::collections::HashMap;

// case1: 
let mut scores = HashMap::new(); 
scores.insert(String::from("blue"), 10);

// case2: 在一个由键值对组成的元组动态数组上使用collect方法
let teams = vec![String::from("Blue"), String::from("Green")]; 
let initial_scores = vec![10, 50]; 
let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect(); 
```

### 哈希映射与所有权

对于实现了Copy trait类型，如i32，当插入的时候，值会被简单地复制到里面

而对于String持有所有权的，值会被转移，且所有权转移给哈希映射

### 访问哈希映射中的值

```rust
let team_name = String::from("Blue"); 
let score = scores.get(&team_name); // 返回的是一个Option<&V>

// 我们也可以用for循环来遍历
for (key, value) in &scores {
  println!(xxxx);
}
```

### 更新哈希映射

当我们对同一个键重复插入，那先前的值会被替换

```rust
scores.insert("blue", 10); 
scores.insert("blue", 25); // 现在blue对应的值为25，而不是10 
```

另外一种场景则是：只在键没有对应值时插入数据

```rust
let mut scores = HashMap::new(); 
scores.entry(String::from("Yellow")).or_insert(50); 
scores.entry(String::from("Blue")).or_insert(50); 
```

entry的or_insert方法定义为返回一个Entry键所指向值的可变引用