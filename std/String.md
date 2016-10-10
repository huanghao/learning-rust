Rust标准库: [std::string](https://doc.rust-lang.org/std/string/)

UTF-8编码自动增长的字符串。这个类是最常见的对其内容字符串有所有权的类型。它和原生`str`有很多联系。

使用`String::from`从一个字符串字面量创建一个`String`

# UTF-8

`OsString` 。。。

# Deref 不知道是啥

因为`String`实现了`Deref<Target=str>`，所以它继承了所有`str`的方法。另外，通过在`String`的前面加一个地址符`&`，它可以传给任何接受`&str`的函数。这样会自动创建一个`&str`，然后传递给函数参数。这种转换的成本是很小而且很普遍，一般情况下函数都会接受一个`&str`，除非有特别的原因才会用`String`当参数。

# 表现形式

一个`String`由三个部分构成：指向字节的指针，长度和容量。指针指向一个内存缓冲区。缓冲区里存放着数据。长度是缓冲区里当前使用的字节数。容量是缓冲区的大小。所以长度总是小于等于容量。

这个缓冲区总是存放在堆上。

通过`as_ptr()`, `len()`和`capacity()`来得到三个部分。

当容量够用的时候，数据不会重新分配。容量不够的时候，会自动扩容。

# 方法

## 构造

* `new()` 创建一个空字符串，不分配任何空间
* `with_capacity(cap: usize)` 创建一个空字符串，分配`cap`的容量
* `from_utf8(vec: Vec<u8>)` 由一个utf8字节向量来创建字符串。函数会检查是否是合法的utf8编码，如果不是会返回错误。为了性能考虑，不会拷贝向量。如果调用者能保证utf8一定是合法的，可以调用不做安全检查的版本`from_utf8_unchecked`
* `from_utf8_lossy(v: &[u8])` 如果遇到不合法的utf8字节，会替换为`�`
* `from_utf16(v: &[u16])`
* `from_utf16_lossy(v: &[u16])`
* `unsafe from_raw_parts(buf: *mut u8, length: usize, capacity: usize)` 由指针，长度，容量三个部分来创建字符串。非常不安全：1）`ptr`必须是由标准库中相同的内存分配器已分配的地址。2）`length`必须小于或等于`capacity`。3）`capacity`的大小一定要和分配的大小对应。如果不满足这几个条件，可能会打乱分配器的内部状态，出现不可知的错误。`ptr`的所有权会转移给创建的字符串，这个字符串可能会回收，重新分配或者修改其中的数据。这样就确保在创建了这个字符串之后，没有其他任何地方可以使用这个指针。

## 转成其他类型

* `into_bytes() -> Vec<u8>` This consumes the String, 所以不需要拷贝其中的内容
* `as_str() -> &str` 字符串切片
* `as_mut_str() -> &mut str` 可变的字符串切片
* `as_bytes() -> &[u8]` 返回一个字节数组
* `unsafe as_mut_vec() -> &mut Vec<u8>` 返回一个可变的vector
* `into_boxed_str() -> Box<str>`

## 查询操作

* `capacity() -> usize` 容量大小
* `len() -> usize` 长度
* `is_empty() -> bool` 是否为空


## 修改大小

* `reserve(additional: usize)` 确保在容量上至少保留`additional`大小。调用以后，可能会增加多于`additional`的空间，来防止再次重新分配。如果不想要“至少”而是“正好”的话，可以用`reserve_exact()`。除非你能做得比分配器更好，一般情况下用`reserve`就足够了。
* `shrink_to_fit` 收缩容量到和长度一样大。正好装下现有的数据

## 修改内容

* `push(ch: char)` 在末尾添加一个字符
* `pop()` 从末尾去掉一个字符。如果字符串为空，就返回None
* `push_str(string: &str)` 在末尾添加一个字符串
* `truncate(new_len: usize)` 把长度截断到`new_len`，如果`new_len`大于实际的大小，将没有作用。如果`new_len`不在utf8的字符边界函数会panic
* `remove(idx: usize) -> char` 删除位置为`idx`的字符并返回。这个操作的复杂度是O(n)。如果`idx`大于字符串长度，或者不在字符的边界上，函数会panic
* `insert(idx: usize, ch: char)` 把字符`ch`插入到位置`idx`上。时间复杂度也是O(n)。如果`idx`大于字符串长度，或者不在字符的边界上，函数panic
* `clear()` 字符串内容清空，容量大小不变。
* `drain<R>(range: R) -> Drain where R: RangeArgument<usize>` 用一个范围来删除这部分内容并返回






