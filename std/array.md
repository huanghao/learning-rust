Rust原生数组：[std::array](https://doc.rust-lang.org/std/primitive.array.html)

大小固定的数组，表示为 `[T; N]`。元素的类型为`T`，大小为`N`。`N`是一个在编译阶段就确定的常数。

初始化的时候，可以把所有的元素都列出来：`[x, y, z]` 或者用相同的值初始化：`[T; N]`，这种形式要求`T`实现`Copy`。当`T`是`Copy`的时候，整个数组`[T; N]`也就是`Copy`。

数组大小在32（包含）以下的会实现以下特性：`Clone`, `Debug`, `IntoIterator`, `PartialEq`, `PatialOrd`, `Ord`, `Eq`, `Hash`, `AsRef`, `AsMut`, `Borrow`, `BorrowMut`, `Default`。

数据可以自动转换成切片`[T]`，所以切片的方法也可以被数据调用。

