# Rust标准库: [std::str](https://doc.rust-lang.org/std/primitive.str.html)

这个库里最重要的类型就是原生字符串类：`str`。也叫做String切片('string slice')。字符串字面量就是这个类型。

这个模块还包含一些辅助的结构体，和字符串模式`pattern`这个子模块。

`str`总是合法的UTF-8。两种特别常见的形式，`&str`, `&'static str`。第二种形式的字符串存储在最终的二进制文件中，在`'static`生命周期中有效。

`&str`包含两个部分，第一个部分是指针，指向一些字节的位置；另一个部分是一个长度。利用`.as_ptr()`和`len()`可以看到这两个部分。

## 方法

### fn len(&self) -> usize

字节长度，而不是字符个数。

### fn is_empty(&self) -> bool

字节长度为0时，返回真

### fn is_char_boundary(&self, index: usize) -> bool

检查index位置是不是utf8字符的边界（开始或结束，开始为包含，结束为下一个）。index为0和self.len()返回真。index大于self.len()返回假。

### fn as_bytes(&self) -> &[u8]

转换成字节切片

### fn as_ptr(&self) -> *const u8

转换成原始的指针。字符串切片其实就是一段字节码，或者原始指针的类型指向第一个字节的位置，类型是`u8`

### unsafe fn slice_unchecked(&self, begin: usize, end: usize) -> &str

从另一个字节切片来创建，跳过安全检查。从begin到end，但不包含end。

安全：调用有责任来保证下面的三个前提
* `begin`要先于`end`
* `begin`和`end`都必须在字符串切片的范围内
* `begin`和`end`都必须在utf8字符的边界上

### unsafe fn slice_mut_unchecked(&mut self, begin: usize, end: usize) -> &mut str

得到一个可变的字符串切片。其他用法和安全检查都和不可变的那个类似。

### fn split_at(&self, mid: usize) -> (&str, &str)

从`mid`位置把一个字符串分成左右两个。`mid`必须在utf8字符的边界上。
当`mid`不是字符边界，或者超过了字符串的末尾，函数会panic

### fn split_at_mut(&mut self, mid: usize) -> (&mut str, &mut str)

拆分成两个可变的字符串。其他和上面的类似。

### fn chars(&self) -> Chars

返回一个字符的迭代器，其中的每个字节都是一个`char`

`It's important to remember that char represents a Unicode Scalar Value, and may not match your idea of what a 'character' is. Iteration over grapheme clusters may be what you actually want.`

    let y = "y̆";
    let count = y.chars().count();
    println!("{}", count);      // 2
    for i in y.chars() {
        println!("{:?}", i)     // 'y'
    }                           // '\u{306}'

### fn char_indices(&self) -> CharIndices

返回一个迭代器，每个元素都是一个二元组，（对应的字节位置，`char`）

    let y = "你们好";
    let count = y.chars().count();
    println!("length: {}", count);
    for (i, c) in y.char_indices() {
        println!("{}: {}, {:?}", i, c, c)
    }

    length: 3
    0: 你, '\u{4f60}'
    3: 们, '\u{4eec}'
    6: 好, '\u{597d}'

### fn bytes(&self) -> Bytes

返回一个bytes的迭代器

### fn split_whitespace(&self) -> SplitWhitespace

用空白分割字符串，并返回包含子字符串的迭代器

### fn lines(&self) -> Lines

用回车换行分割，每行字符串为一个元素，返回迭代器

### fn encode_utf16(&self) -> EncodeUtf16

返回一个utf6编码的迭代器，每个元素都是一个`u16`类型

### fn contains<'a, P>(&'a self, pat: P) -> bool where P: Pattern<'a>

包含指定的子模式`P`就返回真。`P`需要实现`Pattern`特性。字符串就满足这个条件。

### fn starts_with<'a, P>(&'a self, pat: P) -> bool where P: Pattern<'a>

是否以某个模式作为前缀

### fn ends_with<'a, P>(&'a self, pat: P) -> bool where P: Pattern<'a>, P::Searcher: ReverseSearcher<'a>

是否以某个模式作为后缀

### fn find<'a, P>(&'a self, pat: P) -> Option<usize> where P: Pattern<'a>

返回模式第一次出现的位置。不存在就返回None。这个模式可以是`&str`, `char`或者判断字符是否匹配的闭包。

### fn rfind('a, P)(&'a self, pat: P) -> Option<usize> where P: Pattern<'a>, P::Searcher: ReverseSearcher<'a>

返回匹配了模式的最后位置

### fn split<'a, P>(&'a self, pat: P) -> Split<'a, P> where P: Pattern<'a>

使用`P`来切分字符串，返回一个迭代器

### fn rsplit<'a, P>(&'a self, pat: P) -> RSpilt<'a, P> where P: Pattern<'a>, P::Searcher: ReverseSearcher<'a>

上面的返回来

### fn split_terminator<'a, P>(&'a self, pat: P) -> SplitTerminator<'a, P> where P: Pattern<'a>

和`split()`类似，当最后的子字符串为空的话，会被忽略。
当字符串由`P`结束，而不是分隔的时候，使用这个函数。

### fn rsplit_terminator<'a, P>(&'a self, pat: P)

上面那个反过来

### fn splitn<'a, P>(&'a self, count: usize, pat: P) -> SplitN<'a, P> where P: Pattern<'a>
和split类似，但是最多返回`count`个元素。最后一个元素，包含了字符串剩下的部分。

### fn rsplitn

上面那个反过来

### fn matches<'a, P>(&'a self, pat: P) -> Matches<'a, P> where P: Pattern<'a>

返回一个迭代器，每个元素是字符串的一个部分，并且满足模式

`  
    "1abc2abcd3".matches(char::is_numeric)
    ["1", "2", "3"]
`

其他几个看名字猜

### fn rmatches
### fn match_indices
### fn rmatches_indices

### fn trim(&self) -> &str
### fn trim_left
### fn trim_right

去掉空白。
左(left)的意思是指字节数组的开头位置。对于阿拉伯语或者希伯来语这样的从右至左的文字，这里的'left'指的是书写位置右边的文字，而不是书写位置上的左。

### fn trim_matches<'a, P>(&'a self, pat: P) -> &'a str where P: Pattern<'a>, P::Searcher: DoubleEndedSearcher<'a>
### fn trim_left_matches
### fn trim_right_matches

去掉前缀和后缀里连续匹配模式`P`的部分

### fn parse<F>(&self) -> Result<F, F::Err> where F: FromStr

把一个字符串解析成另一个对象。这个对象类型要求实现`FromStr`特性。因为这个函数太通用了，编译器在做类型推导的时候可能会遇到问题。所以，使用这个函数的时候是唯数不多的，需要使用'turbofish': `::<>`语法的情况。使用这个语法来帮助编译器找到正确的类型。

> ::<> 这个符号长得像一条尾巴是涡轮喷气的鱼

> let four = "4".parse::<u32>();
> 为啥不是 "4".parse<u32>() ?

### fn replace<'a, P>(&'a self, from: P, to: &str) -> String where P: Pattern<'a>

替换`from`成`to`。创建了一个新的`String`，然后拷贝了需要的数据过去。

### fn to_lowercase(&self) -> String
### fn to_uppercase

### fn into_string(self: Box<str>) -> String

把一个Box<str>变成String，不拷贝数据，也不分配空间。
TODO: Box是个啥还不明白
