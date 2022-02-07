# chapter4 认识所有权

### 栈与堆

所有存储在栈中的数据都必须拥有一个已知且固定的大小。对于那些在编译器无法确定大小的数据，我们只能存储在堆中，操作系统会根据你的请求在堆中找到一块足够大的可用空间，将地址返回给我们。

向栈上推入数据比在堆上进行分配更有效率，因为操作系统省去了搜索新数据存储位置的工作。

由于多了指针跳转的环节，所以访问堆上的数据要慢于访问栈上的数据

### 所有权规则

- Rust中的每一个值都有一个对应的变量作为它的所有者
- 同一时间内，值有且仅有一个所有者
- 当所有者离开自己的作用域，它持有的值就会被释放

### 变量作用域

```rust
{
  let s = "hello"; 
  // 执行s相关操作
} // 离开作用域，s不可用
```

### String类型

我们使用字符串类型String，它能够处理在编译时未知大小的文本

```rust
let mut s = String::from("hello"); 
s.push_str(", world!");
```

### 内存与分配

对于字符串字面值而言，由于我们编译时就知道其内容，所以硬编码的部分直接嵌入到最终的可执行文件中。

而String类型是可变的，意味着：

- 使用的内存是由操作系统在运行时动态分配出来的
- 使用完需要把内存归还系统

而对于第二步，某些语言自带GC垃圾回收机制，而有些则是纯手动显式释放。

Rust则是利用作用域形成了一套RAII

**内存会自动地在拥有它的变量离开作用域后进行释放**

Rust在变量离开作用域时，会调用一个叫作drop的特殊函数，相关类型的作者可以在drop函数编写释放内存的逻辑

### 移动

在其他代码的拷贝/赋值行为，在Rust则是一种移动语义

```rust
let s1 = String::from("hello"); 
let s2 = s1; 
```

这里等价于s2管理原来s1的对象（浅拷贝），同时让s1失效

### 克隆

克隆对应的就是真正的拷贝，会将原对象的数据一个个拷贝

```rust
let s1 = String::from("hello"); 
let s2 = s1.clone(); 
```

### 栈上数据复制

类似的行为，对于栈上的数据则是一种拷贝

```rust
let x = 5; 
let y = x; // x，y均有效
```

这部分数据完整地存储在栈中，对于这些值的复制操作永远够快，因此不考虑深浅拷贝

Rust提供了一个名为Copy的trait，一旦某种类型拥有这种trait，则他的变量赋给其他变量之后仍保持可用性

如果一个类型或者该类型的任意成员实现了Drop这种trait，那么Rust就不允许其存在Copy

### 所有权与函数

```rust
fn takes_ownership(some_string: String){
  println!("{}", some_string); 
} // some_string离开了作用域，被释放内存

fn makes_copy(some_integer: i32){
  println!("{}", some_integer); 
}

fn main(){
  let s = String::from("hello"); 
  takes_ownership(s); // s的值被move进了函数，从这里开始不再有效  
  
  let x = 6
  makes_copy(x); // x是整型，因此是拷贝，x还是有效
}
```

### 返回值与作用域

```rust
fn gives_ownership() -> String {
  let some_string = String::from("hello"); 
  some_string; // some_string作为返回值移动至调用函数
}

// 取得一个String的所有权并将它作为结果返回
fn takes_and_gives_back(a_string: String) -> String{
  a_string; 
}

fn main(){
  let s1 = gives_ownership(); 
  let s2 = String::from("hello"); 
  let s3 = takes_and_gives_back(s2); 
}
// 1. s3离开作用域被销毁
// 2. 此前s2被移动到函数takes_and_gives_back里，所以无事发生
// 3. s1最后离开作用域并销毁
```

### 引用与借用

Rust允许我们在不转移所有权的前提下，创建一个指向其他值的引用

```rust
fn calculate_length(s: &String) -> usize{
  s.len() 
}

fn main(){
  let s1 = String::from("hello"); 
  let len = calculate_length(&s1); 
}
```

注意这里传参是`&s`形式，参数里的类型注解是`&String`	

由于引用不持有值的所有权，所以离开当前作用域时，指向的值也不会被丢弃

**引用默认不可变**

### 可变引用

只需加入`mut`关键字即可

```rust
fn change(some_string: &mut String){
  some_string.push_str(", world"); 
}

fn main() {
  let mut s = String::from("hello"); 
  change(&mut s); 
}
```

- **对于特定作用域中的特定数据，一次只能声明一个可变引用**

- 不能在拥有不可变引用的同时创建可变引用
- 同时存在多个不可变引用是合理合法的（因为对数据的只读操作不会影响到其他数据）

数据竞争：

- 两个或两个以上的指针同时访问同一空间
- 其中至少有一个指针会向空间中写入数据
- 没有同步数据访问的机制

数据竞争会导致未定义的行为，Rust利用编译器让存在数据竞争的代码无法编译通过

### 悬垂引用

下面是一个悬垂引用的错误示例

```rust
fn dangle() -> &String { // 返回一个String引用
  let s = String::from("hello"); 
  &s // 将指向s的引用返回给调用者
} // 变量 s 离开作用域被销毁，而此时引用只想的内存无效

fn main(){
  let ref_to_nothing = dangle(); 
}
```

### 切片

这是种特殊的引用形式

首先我们编写一个返回字符串首个单词索引的函数：

```rust
fn first_word(s: &String) -> usize {
  let bytes = s.as_bytes(); 
  
  for (i, &item) in bytes.iter().enumerate() {
    if item == b' '{
      return i; 
    }
  }
  return s.len(); 
}
```

但是这个程序和s本身没有产生关联，即你前一次计算好了以后，如果s发生了变化，你的first word并不会变

#### 字符串切片

字符串切片是一种指向String对象某个连续部分的引用

```rust
let s = String::from("hello world"); 

let hello = &s[0..5]; 
let world = &s[6..11]; 
```

[start_index..end_index]

end_index是切片终止位置的下一个索引值

一个语法糖是如果你范围开始于第一个元素，则可以省略0。 如果截止在最后一个元素，也可以省略

```rust
// case1
let slice = &s[0..5]; 
let slice = &s[..5];
// case2 
let slice = &s[0..]; 
let slice = &s[0..len]; 

s[..] 表示整个字符串
```

字符串的切片类型记作`&str`

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes(); 

    for(i, &item) in bytes.iter().enumerate() {
        if item == b' '{
            return &s[0..i];
        }
    }
    return &s[..]; 
}

fn main() {
    let s: String = String::from("Hello world!"); 
    let word = first_word(&s); 
    println!("{}", word); 
}
```

- 字符串字面量就是切片
- 字符串切片作为参数

```rust
fn first_word(s: &str){
  //...
}

first_word(&my_string[..]); 
```

其他类型的切片也类似，这里不再赘述