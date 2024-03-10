
### Conditional Type
可选的方案有enum和union类型，但是enum有运行时开销，union使用需要unsafe。
```rust
trait Proxy<const IS_PTR_GT_64: bool> {
	type Usize64;
}

impl Proxy<true> for () { type Usize64 = usize; }

impl Proxy<false> for () { type Usize64 = u64; }

const IS_PTR_GT_64: bool = sizeof::<usize>() > sizeof::<u64>();

let x: <() as Proxy<IS_PTR_GT_64>>::Usize64 = 0;
// or
type Usize64 = <() as Proxy<IS_PTR_GT_64>>::Usize64;
```

