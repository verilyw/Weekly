# Rust基础知识拾遗



### type

基本类型

+ 数值类型
  + 整数和浮点，如`i8/u8`, `i32/u32`, `f32`
+ 布尔，字符，单元类型`()`
  + `main()`的返回值就是`()`。与之相对应的发散函数指在rust中没有返回值的函数。

(1)类型转换必须是显示的

(2)数值上可以使用方法，如`3.14_f32.round()`，这里用到了类型后缀，因为编译器要知道具体的类型。

### 表达式与语句

+ rust中的函数是由表达式来返回值
+ 基于表达式是函数式语言的重要特征--表达式总要返回值
+ 故从返回值的角度看待语句和表达式
  + 语句：完成一个具体操作，无返回值
  + 表达式：进行求值，然后返回值

### Reference & move sematic // ownership

rust的reference在传参数的时候要对实参添加`&`，感觉这里更像c语言的形式（取地址）;而不像c++中直接在形参中体现引用类型。

rust的reference可分为两种：

+ borrow(不可变)
+ reference(可变)

任何时候，只能有一个可变引用，或者多个不可变引用

引用必须总是有效的

#### 所有权与移动语义

大多数时候理解成c++中的移动语义没什么问题。只是注意的一点是，所有权转移的的时候，如果是栈上的一些基础类型，一般都实现了`COPY`，也就是在赋值的时候并不是移动语义，转交所有权，而是拷贝了一份新的。



### Slice Type

> slice is a kind of reference, it dose not have ownership.
>
> 即它引用collection中一段连续的元素序列

为什么要出现这种类型，主要是与Colletion(e.g 字符串)相关联。在参考书中提到，以字符串为例，如果要获得一个字符串中的一个单词，我们返回其位置，然而这个值与原字符串无关，所以当我们将字符串释放清空，而这个值也还在，只是变成无效了。

```rust
let mut s = String::from("hello world");
let word = first_word(&s);
// println!("{}", word);
s.clear() // error, 
// 因为此处clear需要一个可变引用，但是在之前还存在一个不可变引用word, 因此造成了错误
// 如果在clear之前添加一条使用word的语句，那么则可以顺利编译
```



### Struct

1. definition
   + 关键字
   + 名称
   + 字段
2. instance
   + 每个字段要进行初始化
   + 不需要与定义时顺序一致
   + **从其他instance实例化（要注意的是内部字段可能会发生所有权转移）**
3. access
   + 不变：只有`mut`的instance才能修改字段
   + 不可变

**Misc**:

+ 元组结构体（Tuple Struct）
+ 单元结构体（Unit-like Struct）

关于Struct Data所有权(ownership)----如果字段中采用引用类型，涉及到lifetime的问题

### Method

1. defining method
   + `impl`block for struct type(e.g. Rectangle)
   + `impl`block里面的所有东西都与对应的 struct type相关联
   + `self`与`Self`区别，`self`表示instance，`Self`表示instance的类型，故在方法中传入`&self`实际上是`self: &Self`
   + automatic referencing behaivor
2. Associatied Function

所有定义在impl block中的function（注意与method的区别）都叫做associated function,因为它们都与类型名有关

定义：没有self做为参数。通常用作constructor，用::访问。



### Enum vs. Struct

+ defining

  + 类型名

  + 成员 --> 任何类型

+ 枚举是一个类型，它包含所有可能的枚举成员。枚举值是该类型中具有某个成员的实例。

+ 任何类型都可以放入枚举成员

1. 同一化类型

2. Option处理空值

   ```rust
   enum Option<T> {
       Some(T),
       None,
   }
   ```

如果需要使用Option<T>的值，使用`match`来进行处理。

### enum and match

match is useful and powerful control flow construct.

```rust
match value {
    pattern => expression,
    pattern => expression,
}
```

+ pattern that bind to values
+ match are Exhaustive，穷尽所有情况
  + catch-all patterns and `_`Placeholder

+ concice control flow --- `if let`
+ destructing to Break Apart Values
  1. structs
  2. enum
  3. nested struct and enum
  4. struct and tuple
+ ignore values in a pattern
  + entire value
  + parts of a value, `_`
  + unused variable, `_xx`
  + remaining parts, `..`

+ multiple patterns
  + `v1 | v2`
  + `x..=y`

## Module system

> Why Module?
>
> Manage growing projects with packages, creates and modules

+ Package -- A cargo feature that let you build, test and share crates.
  + `.toml`file that describes how ot build crates
+ Crates  -- A **tree of modules** that produces a lib or execute.
  + library crate
  + binary crate  -- has "main" function, can run.
+ Modules and use -- let you control the orgnization, scope, privace of paths.
+ Path  -- A way of naming an item. such as a struct, function, module.

 ## Collcetion

+ Vector
+ Hash Map
+ String

1. Vector

   1. 创建
      1. use associated function, `Vec::new()`
      2. macro, `vec!`
   2. 更新
      1. push
   3. 访问
      1. 下标
      2. `get`, 返回`Option<&T>`,故要额外的match来解构值
      3. 当对vec进行修改的时候，注意<u>元素引用</u>*（借用）
2. HashMap

   1. 创建

      + `HashMap::new()`, then insert

      + collet方法和迭代器

        + `let team_map: HashMap<_, _> = team_list.into_iter().collect();`
        + 这里注意要指定`HaspMap<_, _>`类型
      
   2. 所有权转移

        + 类型实现Copy特征，无所谓所有权
        + 类型未实现Copy特征，转移给HashMap
        + 若使用引用类型放入HashMap中，确保该引用生命周期至少与hash mapy一样长

   3. HashMap中的K和V都具有**内聚性**，即保证各自类型都要相同。

   3. 更新
   
      1. 覆盖已有值， `insert`
      2. 查询key对应值，若不存在则插入新值，存在则不会插入新值
           1. `let count = map.entry(word).or_insert(0);`
           2. `or_insert`返回&mut v, 故可以修改对应值
           3. 使用count引用时，要先进行解引用“*count", 否则出现类型不匹配。
3. String
   1. `format!`
   2. don't support indexing, because of internal representation
   3. `&str` vs. `String`
      1. `to_string`, `to_owned`, `into`
      2. slice, `&s[..]`, `&s[0..3]` indexing by bytes, sometimes, when passing parameter, `&s`(s is String type) will automatically convert to `&str`type
   4. other function, `push_str`, `replace`, etc.

## 错误处理

   + 可恢复错误，一般是操作错误等不影响全局和系统。

     + `Result<T, E>`

       + ```rust
         enum Result<T, E> {
             Ok(T),
             Err(E),
         }
         ```

       + 处理返回的方法

         + `unwrap` and `expect`。成功返回值，失败panic
         + expect更加常用，因为可以带上错误提示

       + 比如用来处理I/O错误等

   + 不可恢复错误，系统性错误。

     + `panic!`
       + 栈展开（default)
       + 直接终止, `#[profile.release]`以及`panic="abort"`
     + 线程panic，终止该子线程。
     + 调用`panic!`会发生什么
       1. 格式化panic信息，`std::panic::any_panic`
       2. panic hook
       3. stack backtrace

   + [Propagating Errors](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#propagating-errors)

     + `？`
     + 需要一个变量来承载正确值

   + map_err



## Generic

> Why generic?
>
> make code flexible, prevent code duplication

+ In struct definition

  + ```rust
    struct Point<T> {
        x: T,
        y: T,
    }
    
    // multiple type
    struct Point<T, U> {
        x: T,
        y: U,
    }
    ```

+ In enum definition

  + `Option<T>`
  + `Result<T, E>`

+ In Method definition

  + ```rust
    impl<T> Point<T> {
        fn x<T>(&self) -> &T {
            &self.x
        }
    }
    ```

+ Bounds

  + working with generic, the type parameters often must use traits as bounds to stipulate what functionlity a type implements.
  + multple bounds
    + `T: Debug + Display`, Debug和Display都是特征。
  + `where`
    + more expressive

## Trait

+ 特征，共享行为

  + `Debug`
  + `Display`

+ defining

  + ```rust
    pub trait <TraitName> {
        // method(function)
        fn func_name(self) -> Self;
    }
    ```

+ implement a trait on a type

  + ```rust
    impl <TraitName> for <TypeName> {
        
    } 
    
    // simple 
    impl Summary for Article {
        
    }
    
    impl Summary for Tweet {
        
    }
    
    ```

+ Default implementation

  + defualt behavior for some of the methods.

+ trait as Parameters

  + `(item: &impl Summary)`

+ Returning types the implement trait

  + we can return some types that implements the "TraitName" trait without naming concret types
  + **However, you can only use it if returning a single type**


## Test

+ how to write

  1. set up state or data
  2. run code you want to test
  3. assert the result are what you expect

  

+ ```rust
  #[cfg(test)]
  mode tests {
      // use super::*
      
      #[test]
      fn test_func() {
          //...
      }
  }
  ```

+ check the result with the `assert!` macro

  + `assert!`
  + `assert_eq!`
  + `assert_ne!`

+ adding cutom failture messages

+ `should_panic` under `#[test]`

  + get helpful message

+ `Result<T, E>` in Test

## Closure

+ closure are functions that can capture the enclosing envirment

+ ```rust
  |val| val + x
  
  // val: input variable
  // x: capture x variable
  ```

  + `||`, using `||` instead of `()` around input variables
  + optinal body delimination({}) for single expression
  + the ability to capture the other envirnment variables.

+  Captures

  + by reference, `&T`
  + by mutable reference. `&mut T`
  + by value. `T`

+ 类型推导

  + 当编译器推导一种类型，他就会一直使用该类型

+ `Fn`特征

  + `FnOnce`, 会拿走捕获变量的所有权（注：实现`Copy`特征的其实是拷贝）
  + `FnMut`, 以可变借用捕获环境中的值
  + `Fn`, 以不可变借用捕获环境中的值

+ **一个闭包实现那种`Fn`特征取决于该闭包如何使用被捕获变量，而不是取决于如何捕获它们**

  + `move`强调闭包如何实现捕获

+ 一个闭包并不仅仅实现某一种`Fn`特征。它是链式的。

+ 闭包作为返回值

  + Rust要求函数的返回参数和返回类型必须有固定内存大小
  + `Box<dyn Fn(i32)->i32>`

## Iterator

+ 惰性初始化

+ next method

  + ```rust
    pub trait Iterator {
        Type Item;
        fn next(&mut self) -> Option<Self::Item>;
    }
    ```

+ Adaptor

  + consuming adaptor
    + 调用`next`
    + 返回一个值
    + 消耗迭代器上的元素。如，`sum`，它会拿走迭代器所有权。
    + `collect`, 常用，将一个迭代器中元素收集到指定类型
  + iterator adaptor
    + 返回一个新的迭代器，**实现链式方法调用的关键，如`v.iter().map().filter()...`**
    + **Lazy**, 意味着要一个消费适配器来收尾，最终返回一个值
    + `map`, `zip`, `filter`

+ **Iterator vs. IntoIterator(✳)**

  + Iterator， 迭代器特征，只有实现了它，才能称为迭代器，才能调用`next`.
  + IntoIterator, 强调某一类型(可以是迭代器)如果实现该特征，它可以通过`into_iter`, `iter`等方法变成一个迭代器

+ 创建自己的迭代器 —— 只要为自定义类型实现 `Iterator` 特征即可

  + ```rust
    struct counter {
        count: u32,
    }
    
    impl Counter {
        fn new() -> Counter {
            Counter {
                count: 0
            }
        }
    }
    
    impl Iterator for Counter {
        Type Item = u32;
        fn next(&mut self) -> Option<Self::Item> {
            if self.count < 5 {
                self.count += 1;
                Some(self.count)
            } else {
                None
            }
        }
    }
    ```

  + 其他Iterator特征方法默认实现，因为这些默认方法都是基于next实现的。

+ 迭代器是Rust的零成本抽象之一

  + What you do use, you couldn’t hand code any better



## Box<T>

> point to Data on the heap

+ puting a single value on the heap isn't useful. `let a = Box::new(5))`

+ enabling **Recursive Types* with Boxes*

  + Cons list, (1, (2, (3, Nil)))

    + a cons list contain two item
      + the value of current item
      + the next item
    + it should have definite size. Box<T> to get a Recursive Type with known size

  + ```rust
    enum List {
        Cons(i32, Box<List>),
        Nil
    }
    ```

+ `Box<T>` type is a smart pointer because it implement "Deref" trait

## Thread

+ create a new thread

  + `spawn`
  + with closure
  + using `move` closures, (take ownership)
+ wait

  + join
+ Using massage passing to transfer data

  + channel -- message-sending in rust
    + **transmitter**
    + **receiver**
  + mpsc
    + `let (tx, rx) = mpsc::channel();`
  + channels and ownership transfernce
    + `send()`, take ownership if its parameters
  + send multiple values and seeing receiver waiting
    + clone transmitter
+ Share-state Concurrency, (another way of handling concurrency)

  + mutex

    + using `new()` to create
    + `lock()` to acquire the lock
    + `Mutex<T>` is a smart pointer
  + multiple owership with multiple threads
    + `Rc<T>`, create a refernce counted value
      + it is not safe to share across threads
    + `Arc<T>`, Atomic Reference Type

## Macro

+ 声明式宏： `macro_rules!`

+ 过程宏

  + #[derive]， 派生宏
  + 类属性宏
  + 类函数宏

+ 宏 vs. 函数

  + 元编程
  + 可变参数
  + 宏展开
  + 复杂，难以维护

+ ```rust
  #[macro_export]
  macro_rules! vec {
      ($($x:expr), *) {
          {
              let mut temp_vec = Vec::new();
              ${
                temp_vec.push($x);  
              }*
              temp_vec
          }
      };
  }
  ```



## Type Conversion

* as
* try_into
  * 拥有完全控制而不依赖内置转换
  * 返回一个`Result`
* 通用类型转换
  * `as`和`try_into`只能应用在数值类型上

---

+ Rust also offers traits that facilitate type conversions upon implementation. These traits can be found under the [`convert`](https://doc.rust-lang.org/std/convert/index.html) module.

  The traits are the following:

  - `From` and `Into` covered in `from_into`

  - `TryFrom` and `TryInto` covered in `try_from_into`

  - `AsRef` and `AsMut` covered in `as_ref_mut`

  

+ Furthermore, the `std::str` module offers a trait called [`FromStr`](https://doc.rust-lang.org/std/str/trait.FromStr.html) which helps with converting strings into target types via the `parse` method on strings. If properly implemented for a given type `Person`, then `let p: Person = "Mark,20".parse().unwrap()` should both compile and run without panicking.

