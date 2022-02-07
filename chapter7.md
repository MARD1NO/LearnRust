# Chapter7 使用包管理

- 包(package) 一个用于构建，测试并分享单元包的Cargo功能
- 单元包(crate) 一个用于生成库或可执行文件的树形模块结构
- 模块(module)及use关键字 用于控制文件结构
- 路径(path) 一种用于命名条目的方法，条目包括结构体，函数，模块等

### 包与单元包

我们将Rust编译时所使用的入口文件称为单元包的根节点，它同时也是单元包的根模块

而包则由一个或多个单元包集合而成，附带的配置文件`Cargo.toml`描述了如何构建这些单元包的信息

- 一个包只能拥有最多一个库单元包
- 包可以拥有任意多个二进制单元包
- 包必须存在至少一个单元包

```shell
cargo new my-project
     Created binary (application) `my-project` package
ls my-project 
Cargo.toml      src
ls my-project/src 
main.rs
```

执行new命令，cargo会生成一个包并创建对应的配置文件，**Cargo默认将`src/main.rs`视作一个二进制单元包的根节点**

假设包目录有`src/lib.rs`，则视作**库单元包的根节点**

### 通过定义模块来控制作用域及私有性

```shell
cargo new --lib restaurant
```

在`lib.rs`里：

```rust
mod front_of_house{
    mod hosting {
        fn add_to_waitlist(){}

        fn seat_at_table(){}

    }

    mod serving {
        fn take_order(){}

        fn serve_order(){}

        fn take_payment(){}
    }
}
```

我们使用`mod`关键字来定义一个模块，模块内可以包含其他条目的定义

这里整个模块结构为：

```rust
crate 
 |--front_of_host
    |--hosting
       |--add_to_waitlist
       |--seat_at_table
    |--serving
       |--take_order
       |--serve_oder
       |--take_payment
```

### 用于在模块树中指明条目的路径

```rust
// 这段代码没有加pub暴露路径，所以暂时无法编译
mod front_of_house{
    mod hosting {
        fn add_to_waitlist(){}
    }
}

pub fn eat(){
  // 绝对路径
  crate::front_of_house::hosting::add_to_waitlist(); 
  // 相对路径
  front_of_house::hosting::add_to_waitlist(); 
}
```

rust中的所有条目（函数，方法，结构体，枚举，模块，常量）默认都是私有的，**处于父级模块的条目无法使用子模块的条目，但子模块的条目可以使用它所有祖先模块的条目**

### 使用pub关键字来暴露路径

```rust
mod front_of_house{
    pub mod hosting {
        pub fn add_to_waitlist(){}
    }
}

pub fn eat(){
  // 绝对路径
  crate::front_of_house::hosting::add_to_waitlist(); 
  // 相对路径
  front_of_house::hosting::add_to_waitlist(); 
}
```

### 使用super关键字构造相对路径

super关键字类似于`..`，到达上一层级路径

```rust
fn serve_order() {}

mod back_of_house {
  fn cook() {}
  
  fn fix(){
    cook();
    super::serve_order(); 
  }
}
```

### 将结构体，枚举声明为公共的

将enum设置为pub，那其所有项都是pub

将结构体整体设置为pub，那它的字段仍然是私有的，你需要一个个指定

```rust
pub struct Breakfast {
  pub toast: String, 
  fruit: String, 
}
```

### 用use关键字将路径导入作用域

```rust
mod front_of_house{
  pub mod hosting{
    pub fn add(){}
  }
}

use crate::front_of_house::hosting; 

pub fn eat(){
  hosting::add(); 
}
```

使用相对路径略有不同，需要加一个self

```rust
use self::front_of_house::hosting; 
```

### 创建use路径时的惯用模式

**将函数引入作用域的时候，是使用use引入函数的父模块**

```rust
use crate::front_of_house::hosting; 

hosting::xxx(); 
```

而将结构体，枚举和其他条目引入作用域时，**指定使用完整路径**

```rust
use std::collections::HashMap; 
```

### 使用as关键字提供新名称

```rust
use std::io::Result as IoResut; 
```

### 使用pub use重导出名称

当使用use将名称引入作用域时，这个名称会以私有方式在作用域生效，此时外部代码是访问不到这些名称

我们可以使用：

```rust
pub use crate::front_of_house::hosting; 
```

### 使用嵌套的路径

```rust
use std::cmp::Ordering; 
use std::io;

// 等价于
use std::(cmp::Ordering, io); 
```

### 通配符

如果想要把某个路径的公共条目都导入，可以：

```rust
use std::collections::*; 
```

### 将模块拆分为不同的文件

```rust
// src/lib.rs

mod front_of_house;

pub use crate::front_of_house::hosting; 

pub fn eat(){
  hosting::add_to_waitlist(); 
}
```

```rust
// src/front_of_house.rs
pub mod hosting; 
```

```rust
// src/front_of_house/hosting.rs
pub fn add_to_waitlist(){}
```



