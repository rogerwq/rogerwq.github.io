+++
title = "Rust中Copy和Clone两种Trait的比较"
date = 2024-05-19

[taxonomies]
tags = ["Rust"]
+++

## 从 Rust 的变量所有权说起

Rust 中的所有权规则可以总结为：
- Rust 中的每个值都有一个所有者变量。
- 同一时间每个值仅允许存在一个对应的所有者变量。
- 当超出所有者变量的作用域时，从内存中对应的值将被释放。

因此，以下代码将会由于所有权规则产生编译错误。
```rust
fn main() {
   let s1 = String::from("Hello World!"); // s1 为所有者变量。
   let s2 = s1;  // 字符串值由 s1 传递给 s2，s2 成为新的所有者变量，s1 不再拥有该值。

   println!("s1: {}, s2: {}", s1, s2); // 编译错误：s1 无法访问
}
```

但如果将变量的类型改为整型，则可以成功编译。
```rust
fn main() {
    let i1: i32 = 42;
    let i2 = i1; 

    println!("i1: {}, i2: {}", i1, i2); 
}
```

## 对于不同的类型，相同的代码为什么产生了不同的编译结果？

原因在于 i32 类型默认实现了 Copy Trait，从而使得在表达式`let i2 = i1;`中，实际隐含了复制变量 i1 并赋值变量 i2 的操作。
传递给 i2 的是复制后的新值，i1 依然包含原有的整型值。

按照这一思路对字符串相关的代码进行修改，由于 String 类型实现了 Clone Trait，因此将表达式修改为`let s2 = s1.clone();` 则可以成功编译。

## 并非所有的类型都可以实现 Copy Trait
能够实现 Copy Trait 的类型有：
- 所有的整型：u8，i16，u32，i64 等等
- 布尔型：bool
- 浮点型：f32 和 f64
- 字符型：char
- 仅包含以上类型的 struct, tuple 等

也就是说，Copy 操作可以看作是简单的比特复制。

```rust
#[derive(Clone, Copy)] // 编译通过
struct Point {
    x: i32,
    y: i32
}

#[derive(Clone, Copy)] // 编译无法通过，提示 String 类型没有实现 Copy Trait
struct Student {
    name: String,
    age: u8
}
```

## 不能实现 Copy Trait 的类型

- String 型：String 型的实现包含了一个指向堆的指针，复制这个指针会产生多重释放的问题。
- Vec<T>, 即使 T 是可 Copy 的类型, Vec<T> 也无法实现 Copy。原因与 String 的情况类似。
- &mut T：可变引用本质上具有唯一性，因此也无法实现 Copy。

简言之，涉及到内存分配、释放等复杂操作，或者与所有权规则相悖的情况，都无法实现Copy Trait。

这一点也可以从下面这个例子看出，Rust 编译器表示 Drop 和 Copy 是冲突的。

```rust
#[derive(Clone, Copy)] // 提示编译错误，Copy 和 Drop 冲突。
struct Point {
    x: i32,
    y: i32
}

impl Drop for Point { // 编译器认为 Drop 实现中会定义较复杂的内存操作。
    fn drop(&mut self) { todo!() }
}
```

## Clone 是 Copy 的超集
```rust
#[derive(Copy)] // 编译错误，实现 Copy 的前提是实现 Clone。
struct Point {
    x: i32,
    y: i32
}
```

实现 Copy 的方法有且只有以下两种。
```rust
#[derive(Copy, Clone)]
struct MyStruct;
```

```rust
struct MyStruct;

impl Copy for MyStruct { }

impl Clone for MyStruct {
    fn clone(&self) -> MyStruct {
        *self
    }
}
```
Rust 不允许重新定义 Copy，反之 Clone 的重新定义则是允许的。 

## 总结

- Copy 是隐式的，Clone 是显式的。
- Copy 执行代价小，Clone 可能涉及到复杂的内存操作。
- Copy 不允许重新定义，Clone 则可以重新定义。

## 参考链接
- [The Rust Programming Language: What Is Ownership?](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#what-is-ownership)
- [Trait std::marker::Copy](https://doc.rust-lang.org/std/marker/trait.Copy.html#)
- [Matt Oswalt: Copy and Clone in Rust](https://oswalt.dev/2023/12/copy-and-clone-in-rust/)
- [Understanding Rust disambiguating traits: Copy, Clone, and Dynamic](https://blog.logrocket.com/disambiguating-rust-traits-copy-clone-dynamic/)


