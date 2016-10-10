# Unknown

Vec<T>自增长的可变数组。在堆上分配数据。
[T; size] 大小固定的数组。在栈上分配数据。
&[T] 只读的数组切片。它是数组的一部分。

String 自增长的字符串
str 大小固定的字符串
&str 字符串切片

问题：
- 那&mut str是啥？也会有&mut [T]这种东西吗？

大小自增，内容可修改，本身没有什么关系的两个概念。
当然，如果大小发生了改变，也就隐含了内容发生了修改。
他们组合到一起4种情况，



[u8; size] 和 str 有什么区别？
str实际上是utf8的字节数组，而字节数组对于它包含的内容没有假设。str会检查字节是否符合utf8编码标准，又一堆语义上操作字符串的函数。而字节数组都没有。

mut 可变的是变量还是内容本身？ &str本身应该是不可变的，那&mut str是啥意思？创建一个新字符串，把指针变化？那对象是否都是这样？

OsString: https://doc.rust-lang.org/std/ffi/struct.OsString.html

面向python程序猿的rust入门手册。
相同的地方，不同的地方

Box是啥？

这种编译器指示标记，还有啥，什么语法?
> #[derive(Debug)]
> #[allow(dead_code)]

这种语法？
> &self

array: [T; size]
slice: &[T]

unsafe是个啥？
`// We can re-build a str out of ptr and len. This is all unsafe because
// we are responsible for making sure the two components are valid:
let s = unsafe {
    // First, we build a &[u8]...
    let slice = slice::from_raw_parts(ptr, len);

    // ... and then convert that slice into a string slice
    str::from_utf8(slice)
};`


# 小练习

矩阵转置

# 小技巧

unit type `()`, whose only value is also `()`

intergers defaults to `i32` and floats to `f64`

underscores can be inserted in numeric literals to improve readability, e.g. `1_000` is the same as `1000`, and `0.000_001` is the same as `0.000001`

指示数字字面量的类型，可以用`u32`, `i32`这样的后缀。例如：
> 1u32 + 2, 1i32 - 2

unit struct
> struct Nil;

tuple struct
> struct Pair(i32, i32);