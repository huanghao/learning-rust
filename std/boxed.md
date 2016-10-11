Rust标准库： [std::boxed](https://doc.rust-lang.org/std/boxed/)

一个在堆上分配内存的指针类型。

box写做`Box<T>`，提供了简单的堆分配形式。Box对分配的内容有所有权，在离开作用域后自动释放。

# 例子

    #[derive(Debug)]
    enum List<T> {
        Cons(T, Box<List<T>>),
        Nil,
    }

    fn main() {
        let list: List<i32> = List::Cons(1, Box::new(List::Cons(2, Box::new(List::Nil))));
        println!("{:?}", list);
    }

显示的结果：

`Cons(1, Cons(2, Nil))`

这个递归结构必须用Box类型，如果写成这样`Cons(T, List<T>)`，那么List的长度就无法确定了，就没法决定给`Cons`分配多少内存。如果用Box，那么大小就确定了。

# std::boxed::Box

* `new(x: T) -> Box<T>` 用一个值创建Box
* `unsafe from_raw(raw: *mut T) -> Box<T>` 用一个指针创建Box
* `into_raw() -> *mut T` 把一个Box变成一个指针




