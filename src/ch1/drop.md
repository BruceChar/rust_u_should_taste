# Drop
Rust编译器自动管理内存还有一个功臣，就是Drop。先来感受一下`Rust std::mem::drop`的实现
```rust
pub fn drop<T>(_x: T) {}
```
惊喜不！是不是有种大道至简的感觉。当我第一次看到的时候有点不得其解，随之就是不可思议。感觉Rust所有权的抽象闭环很完美。

那[Drop](https://doc.rust-lang.org/std/ops/trait.Drop.html)是干嘛的呢？是
```rust

```
### Order
#### Rules
	作用域变量按声明的逆序drop
		嵌套作用域由内而外
	函数参数也按申明的逆序drop
	`struct fields` 按申明的顺序drop (代码的顺序，不是内存布局的顺序)
	序列类型，`tuple enum array owned slices` 按 sequence顺序drop

```rust
struct Token(&'static str);
impl Drop for Token {
    fn drop(&mut self) {
        println!("[drop] {}", self.0);
    }
}
fn scope() {
	// 直接drop
    let _ = Token("_ pattern");

    let t1 = Token("1");
    // 无影响
    let _ = t1;

    let t2 = Token("2");
    // 改变申明顺序，导致drop顺序变化，相当于t3
    let t2 = t1;
    println!("t2 re-assigned");
}
```
结果：
```shell
[drop] _ pattern
t2 re-assigned
[drop] 1
[drop] 2
```

#### Expression: match, if let and if
```rust
fn exprs() {
    fn make_token(s: &'static str) -> Result<Token, ()> {
        Ok(Token(s))
    }
    
    match make_token("matched token") {
        Ok(_) => println!("match arm"),
        Err(_) => unreachable!(),
        // 这里drop
    }
    println!("after match");

    if let Ok(_) = make_token("if let token") {
        println!("if let body");
        // 这里drop
    }
    println!("after if let");

    if make_token("if token").is_ok() /* 这里drop */ {
        println!("if body");
    }
    println!("after if");
}
```
结果：
```shell
match arm
[drop] matched token
after match
if let body
[drop] if let token
after if let
[drop] if token
if body
after if
```

#### Function arguments
```rust
fn fn_args() {
    fn takes_args(t1: Token, (_, t2): (Token, Token), _: Token) {
        println!("function body");
    }

    takes_args(Token("t1"), (Token("t2.0"), Token("t2.1")), Token("_"));
}
```
结果：
```shell
[drop] _
[drop] t2.1
[drop] t2.0
[drop] t1
```