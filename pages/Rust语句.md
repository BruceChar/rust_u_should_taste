## 语句(statements)
Rust语句分两种，声明语句和表达式语句
### 声明语句
#### let 语句
```rust
let mut x = 0;
let Some(t) = v.pop() else { panic!(); };
```
#### item声明
- 模块
	`mod rust {}, mod rust;`
- extern
	`extern create pcre;  extern "C" {}`
- use声明
	`use rust::WTF;`
- 类型声明
	 `type Point = (u8,u8);`
- 函数，结构体，枚举，union，trait定义
	`union WTFUnion {w: u32, t: i32, f: f32 }`
- constant、static
	`const THREADS_NUM = 256;`
	`static mut counter: u64 = 0;`
- impl语句
	`impl Trait for Struct {}`
### 表达式语句
`表达式 + ;`就是表达式语句，将表达式的值忽略。
