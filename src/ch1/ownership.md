# Rust的所有权
要清晰认识Rust的所有权，就必须知道值在堆栈中怎么存储和使用的。

#### **栈**

栈是一个自顶向下，资源有限，效率很高的线性存储结构，因为操作只要移动栈指针。栈是程序运行的基础。每当一个函数被调用时，一块连续的内存就会在栈顶被分配出来，这块内存被称为帧。编译器需要知道每个函数调用用到哪些寄存器，栈上存放哪些局部变量，这些都要在编译时确定。

栈是程序运行的基础。每当一个函数被调用时，一块连续的内存就会在栈顶被分配出来，这块内存被称为帧（frame）。存在栈上的内容要能够在编译期确定使用空间的大小。同时函数调用结束，栈帧会被回收，局部变量就会失效。

借用陈天老师的图![[stack_frame.png]]

所以在使用栈时有几个注意点
- 避免在栈上分配大数组
- 避免过深的递归调用
- 想要提高性能可以使用smallvec等在栈上的方案

#### 堆
堆有足够的资源空间，但是效率更低，可以在运行时动态分配内存。所以**不确定大小**的资源和**不确定生存周期**的资源都要分配在堆上，通过指针来访问。而这个指针是存在栈上的。

但是需要手动管理资源的分配和释放，一不注意就会导致内存泄漏，访问非法地址等重大内存安全问题。

#### ownership
好在Rust帮我们处理了，那Rust是怎么做的呢——所有权。

那到底什么是所有权？先看官方的三个原则
- 每一个值都有一个所有者
- 同一时刻每个值只能有一个所有者
- 所有者离开作用域，值会被丢弃

要解释所有权，就要先介绍Rust的两个概念，Move语义和Copy语义。先说简单点的Copy语义，也是大家印象中变量的行为。
#### Copy语义
Copy代表值在移动的时候，是按位拷贝的，原始数据会保留。意味着你将一个变量赋值给另一个变量，原变量还可以继续使用，因为它拥有的值还有效。

原则上在编译期能够确定值存储大小的类型。包括**标量类型**，整数，浮点数，布尔类型，字符类型都是；**切片类型**，字符串切片&str和数组切片&[T]；**函数类型**，就是一个指针；**组合类型**，数组和元组。

还有**复合类型** enum和struct，如果复合类型的元素都是Copy的，则可以通过`#[derive(Copy)]`来自动派生成Copy类型。

#### Move语义
名字已经说的很清楚了，就是把值给移走。原先的变量就成了未定义状态，不符合RALL原则，所以使用就是非法的了。

Rust默认是Move语义，除非类型是Copy的，就像你必须显示的`derive(Copy)`，复合类型才会变成Copy，不然就是Move的。

看组例子
```rust
fn main() {
	// Copy 
	let x = 0; // x是u32,在栈上，是Copy的
	let y = x; // x copy to y
	let z = x + 1; // Ok

	// Move
	let x = Box::new(0); // x用Box分配在堆上，是Move的
	let y = x; // x move to y
	let z = *x + 1;  // Error: use moved value x

	// Derive Copy
	let r = RGB(0,0,0);
	let g = r; // Move还是Copy取决于RGB是什么类型
	let b = r; // 如果注释掉下面的#[derive(Clone, Copy)]，将出错
}

#[derive(Clone, Copy)]
struct RGB(u8, u8, u8);

```
相信看完上面的例子，对Move还是Copy已经有比较清晰的认识了。
#### Borrow
讲完Move和Copy还有一个不得不提的就是借用(Borrow)。

**Rust中的引用就是借用**。是为了区别其他语言中引用提出的新概念。
其他语言中引用是别名，是值的地址。多个引用可以对值无差别共享。
但Rust中的引用是有区别的，因为rust的所有权机制，你只能从拥有者那临时获得使用权，
默认是只读不可变引用`&T`，可变引用需要显示声明 `&mut T`。
就像借书，只能翻看，没有允许你是不可以乱涂乱画的，最终是要还的。

Rust中有两种引用：不可变引用 `&T` 和可变引用 `&mut T`。

可以把借用当成rust对引用的独特实现，是一种行为。
而把引用当成一种表述，一种类型。
#### 编译期读写锁
Rust是一门自带读写锁的语言，为什么这么说。因为Rust的所有的变量都是有严格读写权限的。默认是只读的，如果想要改变变量，必须显示声明为`mut`。
对于引用
- 任一时刻，要么持有任意数量不可变引用，要么只有一个可变引用
-  引用必须是有效的

```rust
fn main() {
	let x = 0;
	let y = &x;
	// 下面代码会报错，因为y持有了对x的引用
	x += 1;
	// x的引用持续到这，在这之前不能更改
	println!("{x} {y}");
}

fn dangle_ref() -> &str {
	let s = "danger".to_string();
	// 无效引用，因为返回后s被释放了
	// rust编译器会拒绝这类代码
	return &s
}
```

回过头再看Rust所有权的规则，是不是就能理解了。
#### &T 和 &mut T
`Rust`的引用和其他语言的都不一样，因为它有读写锁。特别是对于`&mut T`，程序中大多数问题都和它有关。因为`&T`是`Copy`的，而`&mut T`不是。
```rust
fn main() {
	let mut x = 0;
	let y = &mut x;
	let w = y;
	*w += 1;
	// Error: use of moved value y
	println!("{w} {y}");

	let x = 0;
	let y = &x;
	let w = y;
	// Ok: y is Copy
	println!("{w} {y}");
}
```
#### Rust是显式语言
Rust是一门显式语言，意味着它要求代码的行为是确定的，任何有代价，有风险的都需要明确的显示出来。

比如你想要修改一个变量，那就必须指明这个变量是可变的。
如果你要拷贝一个值，必须`clone`调用，告知你这个操作很可能代价很大。

