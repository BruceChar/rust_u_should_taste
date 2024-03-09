### if else
和其他语言一样，Rust条件判断使用`if {} else if {} else` 语法。
不一样的是，Rust的条件是表达式，意味着每一个条件都有一个返回值，且各个分支的返回值类型必须一样。
```rust
let x = 0;
if x > 0 {
	return 1;
} else if x == 0 {
	// Error: expected i32
	return "zero"
} else {
	return -1;
}
```
### 循环
Rust支持三种循环`loop, while, for`。`loop`语义上和`while true`一样，但是需要用到无限循环时，推荐都是用loop，忘掉while true吧
```rust
{
	let x;
	// ok with loop
	loop {
		x = 1;
		break;
	}
	println!("{x}");
}

{
	let x;
	while true {
		x = 1;
		break;
	}
	// Error: `x` used here but it is possibly-uninitialized
	println!("{x}");
}
```
#### for
Rust的for循环是迭代器，`for item in iterator`。
#### break
Rust的break返回当前层，在loop循环和breakable block中，可以返回值。
```rust
let x = loop {
	break true;
};
```
**in labelled block**
在标签块中，可以使用break返回。标签块中的标签`'block`是必须得。
```rust
let result = 'block: {
    do_thing();
    if condition_not_met() {
        break 'block 1;
    }
    do_next_thing();
    if condition_not_met() {
        break 'block 2;
    }
    do_last_thing();
    3
};
```
**break vs return**
```rust
fn return1() {
    if (return { print!("1") }) {}
}
fn return2() {
    if return { print!("2") } {}
}

fn break1() {
    loop {
        if (break { print!("1") }) {}
    }
}

fn break2() {
    loop {
        if break { print!("2") } {}
    }
}

fn main() {
    return1(); return2();
    break1(); break2();
}
// 121, the break 2, do nothing.
```
#### 循环标签
rust可以给每一层循环一个标签，可以用`break， continue` 来不同层级之间的跳转。如果不同层标签一样，内层的会覆盖外层的。遵循局部变量的卫生性和shadowing规则。
```rust
'outer: loop {
	println!("outer");
	'inner: for i in 0..10 {
		if i % 2 == 0 {
			break 'outer;
		}
	}
}
```

