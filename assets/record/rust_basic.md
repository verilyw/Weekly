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



