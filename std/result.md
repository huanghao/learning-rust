Rust标准库：[std::result](https://doc.rust-lang.org/std/result/index.html)

`Result<T, E>`是一个用来返回值和处理失败的枚举。`Ok(T)`代表成功，而且包含一个值，类型为`T`。`Err(E)`代表失败，包含一个类型为`E`的出错信息。

    enum Result<T, E> {
        Ok(T),
        Err(E),
    }

# Result必须被使用

使用普通的返回值来代表错误，有一个常见的问题。那就是返回值会被忽略，从而错误没有得到处理。Result被标记为`#[must_use]`，如果Result被忽略的话，编译器会产生一个警告。

# `try!`

使用这个宏可以用来简化出错检查代码的写法。

    macro_rules! try {
        ($e:expr) => (match $e {Ok(e) => e, Err(e) => return Err(e)})
    }

从这个宏的定义可以看出。`try!`接受一个表达式，如果表达式返回的是Ok，它会把对应的值取出来返回。如果表达式返回的是Err，它会提前return这个错误。所以`try!`用在返回Result的函数中可以提前退出。
