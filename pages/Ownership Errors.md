#### 悬垂引用
```rust
fn return_dangle() -> &String {
	let s = String::from("WTF");
	&s
}
```
**UNSAFE:** 因为s在栈上，函数返回就被释放了，s的生命周期太短。
**FIX：** 根据实际需求选择方式
```rust
1. 返回所有权
2. 'static 生命周期
3. 使用Rc指针
4. 创建空间替换
fn return_string(out: &mut String) {
	out.replace_range(.., "hello world");
}
```

#### 权限受限
```rust
fn stringify_name_with_title(name: &Vec<String>) -> String {
    name.push(String::from("Esq."));
    let full = name.join(" ");
    full
}
// ideally: ["Ferris", "Jr."] => "Ferris Jr. Esq."
```
**UNSAFE:** 改变不可变引用的内容，可能会导致外部的不可变引用无效，比如Vec重新分配了地址。
**FIX：** 理想方式还是不改变签名，只用不可变引用模式
```rust
1. 改变函数签名为&mut
2. 改函数签名为传值：所有权
3. 不改签名，内部clone
4. 使用函数拼接
fn stringify_name_with_title(name: &Vec<String>) -> String {
    let mut full = name.join(" ");
    full.push_str(" Esq.");
    full
}
```

#### 同时借用可变和不可变引用
```rust
fn add_big_strings(dst: &mut Vec<String>, src: &[String]) {
    let largest: &String = 
      dst.iter().max_by_key(|s| s.len()).unwrap();
    for s in src {
        if s.len() > largest.len() {
            dst.push(s.clone());
        }
    }
}
```
**UNSAFE:** 可变引用dst的操作，可能会导致dst重分配，使largest无效
**FIX：** 使不可变引用和可变引用的使用不重叠
```rust
1. clone
2. 改变执行顺序，先比较，和clone一样，也会分配内存到to_add
fn add_big_strings(dst: &mut Vec<String>, src: &[String]) {
    let largest: &String = dst.iter().max_by_key(|s| s.len()).unwrap();
    let to_add: Vec<String> = 
        src.iter().filter(|s| s.len() > largest.len()).cloned().collect();
    dst.extend(to_add);
}
3. 只使用largest.len()，理想方案
fn add_big_strings(dst: &mut Vec<String>, src: &[String]) {
    let largest_len: usize = dst.iter().max_by_key(|s| s.len()).unwrap().len();
    for s in src {
        if s.len() > largest_len {
            dst.push(s.clone());
        }
    }
}

```

#### Copy 和 Move
```rust
// ok
fn copu() {
	let v: Vec<i32> = vec![0, 1, 2];
	let n_ref: &i32 = &v[0];
	let n: i32 = *n_ref;
}
// error
fn move() {
	let v: Vec<String> = vec![String::from("Hello world")];
	let s_ref: &String = &v[0];
	let s: String = *s_ref;
}
```
**UNSAFE:** 解引用`*s_ref`会使得s拥有String("Hello world")的所有权，从而导致double free。
**FIX:** 非Copy的数据，如果要拥有所有权，要么clone，要么将其从容器中remove出来
**NOTE：** `&mut T`也是非Copy的，哪怕T是Copy的

#### 改变元组、数组中的不同字段
在很多情况下，改变不同字段的程序是安全的，但是目前Rust的borrow checker不够智能，无法分辨出，所以会拒绝我们的代码。
```rust
fn main() {
	let mut a = [0, 1, 2, 3];
	let x = &mut a[1];
	let y = &a[2];
	*x += *y;
}
```
**UNSAFE:**   safe 
**FIX：** split_at_mut,  unsafe raw指针