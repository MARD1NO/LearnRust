# 通用编程概念

### 变量与可变性

`mut` 关键字声明变量是可变的

### 变量与常量的不同

我们使用关键字`const`和类型标注来声明一个常量：

```rust
const MAX_POINTS: u32 = 100_000; 
```

### 隐藏

新声明的变量可以覆盖掉旧的同名变量

```rust
let x = 5; 
let x = x + 1; 
let x = x * 2; // x = 12
```

### 数据类型

#### 标量类型

整数类型有：

```
i8 u8
i16 u16
i32 u32
i64 u64
isize usize 这个是根据程序运行的目标平台决定的，在64位架构上，它们就是64位
```

对于字面量我们可以使用类型后缀，比如57u8，代表u8类型的整数57

浮点数类型有：

```
f32
f64
```

默认推导：

整数类型是i32

浮点类型是f64

布尔类型：bool

字符类型：char，需要使用单引号

#### 复合类型

两种基础复合类型是元组，数组

元组中的元素类型可以是不一样的：

```rust
let tup: {i32, f64, u8} = {500, 6.4, 1}; 
```

(类型注解是不必要的)

我们可以通过模式匹配元组，得到单个值

```rust
let {x, y, z} = tup; 
```

也可以用 . 来访问

```rust
let x0 = x.0; 
let x1 = x.1; 
let x2 = x.2; 
```

数组类型，拥有固定的长度，和相同类型的元素

```rust
let a = [1, 2, 3, 4, 5]; 
```

我们也可以写出数组的类型和元素个数：

```rust
let a : [i32, 5] = [1, 2, 3, 4, 5]; 
```

另一种相似的初始化写法：

```rust
let a = [3; 5]; // 含有5个元素，都是3
```

使用索引访问

```rust
let first = a[0]; 
```

### 函数

- 函数签名需要显式地声明每个参数的类型

```rust
fn func(x: i32){
  // ...
}
```

与语句不同，表达式会计算出某个值作为结果

```rust
let y = {
  let x = 3; 
  x + 1
}; 
```

注意这里 `x+1`没有分号了，意味着这个表达式返回的是 `x+1` 结果，即4

函数的返回值等同于**函数体最后一个表达式的值**，我们可以使用return关键字，也可以隐式返回

```rust
// right!
fn plus_one(x: i32){
  x + 1
}
// wrong!
fn plus_one(x: i32){
  x + 1; // no need ;
}
```

### 控制流

````
if, else if 与python类似
````

### 使用循环

1. loop

```rust
let mut counter = 0; 
let result = loop {
  counter += 1; 
  
  if counter == 10 {
    break count * 2; 
  }
}; // result = 20
```

2. while

```rust
while number != 0 {
  // ...
}
```

3. for循环

```rust
let a = {10, 20, 30, 40}; 
for element in a.iter() {
  println!("The value is {}", element); 
}
```

