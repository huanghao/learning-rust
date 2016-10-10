Rust标准库：[std::vec](https://doc.rust-lang.org/std/vec/)

一个在堆上分配内存，连续增长的数组类型。写做`Vec<T>`，读作`vector`。
向量操作中，随机索引的时间复杂度是O(1)，在尾部添加和删除的渐进复杂度也是O(1)。

可以使用`Vec::new()`或者宏`vec!`来创建一个向量。

这个模块中最重要的类型是`std::vec::Vec`，先来看这个类。

# std::vec::Vec

## 索引

Vec实现了`Index` trait，所以可以使用下标做随机的访问。但是要小心，程序会panic如果下标越界的话。

## 切片

Vec是可以被修改的。相反“切片”是只读的。为了得到一个切片，使用`&`。
一般情况下，如果只需要读，那么传递切片当参数，而不是一个Vec。同样类似的是`String`和`&str`

## 容量和重新分配

和`String`类似，vector也有容量(`capacity`)和长度(`length`)的概念。容量代表已经分配的空间大小。长度代表已经存放在vector上的数据个数。如果长度超过了容量，容量会自动增长，但是包含的元素会被重新分配。重新分配会慢，所以建议使用`Vec::with_capacity`

## 担保

因为Vec实在太基础了，所以在设计会有一些担保。在一般情况下，开销尽量小，而且在不安全的代码中正确的被处理。这里指的是没有被修改过的Vec<T>。例如自定义了分配器，那就不一定能保证了。

底层上，Vec是一个三元组（指针，容量，长度）。不多也不少。三部分的顺序是没有保证的，所以一定要使用对应的函数来修改它们。指针绝不会为空，所以它并没有对空指针的情况做优化。

然而，指针实际上并不是总指向一个已分配的内存。如果构造一个容量为0的Vec，如：`Vec::new()`, `vec![]`，`Vec::with_capacity(0)`或者对一个空数组调用`shrink_to_fit`，不会有地址需要分配。类似的，如果在向量上存放大小为0的类型，也不需要空间。（注意，在这种情况下，Vec的`capacity()`的返回有可能不为0）。Vec只有在`mem::size_of::<T>() * capacity() > 0`的情况下，才会分配内容。一般来说，因为Vec的分配细节太微妙，所以强烈建议，'you only free memory allocated by a Vec by creating a new Vec and dropping it.' 通过创建一个新的vec来释放一个老的vec的空间。（不明觉厉!!）

Vec在堆上分配内存（Rust编译器的默认设置），它的指针指向一块大小为`len()`并且被初始化过的区域。后面跟着一块大小为`capacity() - len()`未被初始化的区域。

有两个原因，当Vec里存放了数据的时候，它绝不会执行“小改进”（small optimization)：
- 这样会让不安全的代码更难正确的操作一个vec。如果它被移动过，很难保证数据在一个稳定的地址上。并且判断一个Vec是否真的分配了空间也变得更困难了。
- 这样会在普通的情况上增加开销，例如每一次访问操作都会增加一些分支。

为避免不必要的分配和回收，Vec绝不会自动的减小容量，即使已经清空的情况下。清空一个vec后又填满，不会有分配操作。当需要释放不需要的空间时，可以调用`shrink_to_fit`函数。

在容量有剩余的时候，`push`和`insert`操作绝不会新分配空间。只有在容量和长度相等的时候，才会。这表示，`capacity()`返回的容量是非常精确的。在必要的情况下，可以根据它来手动进行内存的回收。批量插入可能会导致重新分配空间，甚至在不是必要的时候。

Vec对空间增长的策略没有保证，甚至在调用`reserve`的时候。当前的策略非常的基础，使用一个非常数的增长因子是很有必要的。不管使用什么策略，都会保证`push`的渐进复杂度是O(1)。

使用`vec![x; n]`, `vec![a, b, c, d]`和`Vec::with_capacity(n)`会得到一个容量大小很明确的vec。使用`vec!`宏的情况下，长度和容量相等，Vec可以被转换成一个`Box<[T]>`，而不会重新分配或者移动元素。

对于被移除的元素，vec没有任何保证。在这些没有被初始化的内存上，为了性能或者实现的简单，可能会做任何需要的操作。为了安全的目的，移除数据不一定代表数据会被擦除。即使drop了一个vec，它的空间也可能被另一个vec重新使用。即使你把vec的空间都操作清零了，并不代表这个清零操作一定会被执行，编译器可能会把它优化掉。

vec对drop元素的顺序没有保证。（这个顺序原来变化过，以后也可能再变化）

## 方法

### 构造

* `new() -> Vec<T>`
* `with_capacity(capacity: usize) -> Vec<T>`
* `unsafe from_raw_parts(ptr: *mut T, length: usize, capacity: usize) -> Vec<T>`

### 属性

* `capacity() -> usize`
* `len() -> usize`
* `is_empty() -> bool`

### 修改容量

* `reserve(additional: usize)`
* `reserve_exact(additional: usize)`
* `shrink_to_fit()`

### 其他类型

* `into_boxed_slice() -> Box<[T]>`
* `as_slice() -> &[T]`
* `as_mut_slice() -> &mut [T]`

### 修改内容

* `truncate(len: usize)`
* `unsafe set_len(len: usize)`
* `swap_remove(index: usize) -> T` 用最后一个元素代替`index`位置的元素，然后把它返回
* `insert(index: usize, element: T)`
* `remove(index: usize) -> T`
* `retain<F>(f: F) where F: FnMut(&T) -> bool` 原地操作，保留`f(&e)`为真的元素，移除其他
* `push(value: T)` 尾部添加
* `pop() -> Option<T>` 尾部移除
* `append(other: &mut Vec<T>)` 把`other`里的元素都追加到尾部，并把`other`清空
* `drain<R>(range: R) -> Drain<T> where R: RangeArgument<usize>` 移除`R`范围的元素，并返回
* `clear()` 清空
* `split_off(at: usize) -> Vec<T>` 从`at`位置一份为二。原始的Vec包含`[0, at)`元素，返回的Vec包含`[at, len)`的元素。原始vec的容量不变。
* `resize(new_len: usize, value: T)` 如果`new_len`大于`len()`，Vec用`value`来填充扩大到`new_len`。否则，Vec被截断到`new_len`
* `extend_from_slice(other: &[T])` 拷贝`other`然后追加到末尾
* `dedup()` 去除连续的重复元素





