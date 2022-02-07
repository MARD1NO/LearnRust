# chapter5 结构体

### 定义和实例化结构体

```rust
struct User {
    username: String, 
    email: String, 
    sign_in_count: u64, 
    active: bool,        
}


fn main() {
    let user1 = User{
        email: String::from("xxx.com"), 
        username: String::from("zzk"), 
        active: true, 
        sign_in_count: 1, 
    }; 
}

```

我们可以用点号来访问

一旦结构体定义为是可变的，那所有字段都是可变的，不存在部分可变的情况

### 变量名与字段名相同时，可以简写

当变量名与字段名相同，可以直接写变量名而不用单独写出字段名

```rust
fn build_user(email: String, username: String) -> User {
  User{
    email, 
    username, 
    sign_in_count: 1, 
    active: true
  }
}
```

### 使用结构体更新语法，根据其他实例来创建其他实例

```rust
let user2 = User {
  email: String::from("xx.com"), 
  username: String::from("zzk"), 
  active: user1.active, 
  sign_in_count: user1.sign_in_count
}; 

// 也可以直接
let user2 = User {
  email: String::from("xx.com"), 
  username: String::from("zzk"), 
  ..user1
}; 
```

### 使用不需要对字段命名的元组结构体来创建不同的类型

```rust
struct Color(i32, i32, i32); 
struct Point(i32, i32, i32); 

let black = Color(0, 0, 0); 
let origin = Point(0, 0, 0); 

// 这里虽然Color和Point结构体都是由3个i32组成，但是类型是不一样的，不能在接受Point的函数传入Color结构体
```

### 没有任何字段的空结构体

会用于在某个类型上实现一个trait

### 基于结构体的一段代码

```rust
struct Rectangle{
    width: u32, 
    height: u32, 
  }
  
fn area(rectangle: &Rectangle) -> u32 {
rectangle.width * rectangle.height
}

fn main(){
  let rect1 = Rectangle{width: 30, height: 50}; 
  println!(
      "This area of the rectangle is {} square pixels. ", area(&rect1)
  ); 
}
```

### 通过派生trait来增加实用功能

```rust
# [derive(Debug)]
struct Rectangle{
    width: u32, 
    height: u32, 
}
  
fn main(){
  let rect1 = Rectangle{width: 30, height: 50}; 
  println!("rect1 is {:?}", rect1); 
}
```

### 方法

```rust
impl Rectangle{
    fn area(&self) -> u32{
        return self.width * self.height; 
    }
  
  fn can_hold(&self, other: &Rectangle) -> bool {
        return self.width > other.width && self.height > other.height; 
    }
}
```

使用impl关键字包住相关方法

第一个参数是`&self`，来指代实例本身的引用。由于area函数只需要只读Rectangle的属性，因此这里是不可变引用。当你需要修改实例的东西，则需要`&mut self`

### 关联函数

我们还可以定义不用接收self作为参数的函数，他们不会作用于某个实例，因此称之为函数

```rust
impl Rectangle{
  fn square(size: u32) -> Rectangle {
    Rectangle{width: size, height: size}
  }
}

let rect3 = Rectangle::square(3); 
```

我们可以把方法放在不同的impl块里