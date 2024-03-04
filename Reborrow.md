- Rust conceptually handles re-borrows by maintaining a "borrow stack"
- Only the one on the top of the stack is "live" (has exclusive access)
- When you access a lower one it becomes "live" and the ones above it get popped
- You're not allowed to use pointers that have been popped from the borrow stack
- The borrow checker ensures safe code code obeys this
- Miri theoretically checks that raw pointers obey this at runtime

```rust
fn main() {
	let mut x = 10;
	let r1 = &mut x;
	let r2 = &mut *r1;
	// Error
	*r1 += 1;
	*r2 += 2;
	// Ok
	// *r2 += 2;
	// *r1 += 1;
}
```
如果我们把编译错误的代码用指针替换，Rust编译通过了
```rust
fn main() {
	let mut x = 10;
	let r1 = &mut x;
	let p = r1 as *mut _;
	// Ok
	*r1 += 1;
	*p += 2;
}

/// 13
```
如果我们用miri帮我们检测下
```rust
MIRIFLAGS="-Zmiri-tag-raw-pointers" cargo +nightly-2024-01-23 miri run

    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running cargo-miri.exe target\miri

error: Undefined Behavior: no item granting read access 
to tag <untagged> at alloc748 found in borrow stack.

 --> src\main.rs:9:9
  |
9 |         *p += 2;
  |         ^^^^^^^^^^ no item granting read access to tag <untagged> 
  |                    at alloc748 found in borrow stack.
  |
  = help: this indicates a potential bug in the program: 
    it performed an invalid operation, but the rules it 
    violated are still experimental
```
Miri帮我们捕捉到了潜在的bug行为，这非常Rust！
我们现在看点复杂的,`&mut -> *mut -> &mut -> *mut`

```rust
unsafe {
    let mut data = 10;
    let ref1 = &mut data;
    let ptr2 = ref1 as *mut _;
    let ref3 = &mut *ptr2;
    let ptr4 = ref3 as *mut _;

    // Access the first raw pointer first
    *ptr2 += 2;

    // Then access things in "borrow stack" order
    *ptr4 += 4;
    *ref3 += 3;
    *ptr2 += 2;
    *ref1 += 1;
    println!("{}", data);
}

cargo run
22

MIRIFLAGS="-Zmiri-tag-raw-pointers" cargo +nightly-2022-01-21 miri run

error: Undefined Behavior: no item granting read access 
to tag <1621> at alloc748 found in borrow stack.

  --> src\main.rs:13:5
   |
13 |     *ptr4 += 4;
   |     ^^^^^^^^^^ no item granting read access to tag <1621> 
   |                at alloc748 found in borrow stack.
   |

```

Wow yep! In strict mode miri can "tell apart" the two raw pointers and have using the second one invalidate the first one. Let's see if everything works when we remove the first use that messes everything up:

```rust
unsafe {
    let mut data = 10;
    let ref1 = &mut data;
    let ptr2 = ref1 as *mut _;
    let ref3 = &mut *ptr2;
    let ptr4 = ref3 as *mut _;

    // Access things in "borrow stack" order
    *ptr4 += 4;
    *ref3 += 3;
    *ptr2 += 2;
    *ref1 += 1;

    println!("{}", data);
}

cargo run
20

MIRIFLAGS="-Zmiri-tag-raw-pointers" cargo +nightly-2022-01-21 miri run
20
```

#### Miri
```rust
> cargo +nightly-2024-01-23 miri test
> MIRIFLAGS="-Zmiri-tag-raw-pointers" cargo +nightly-2024-01-23 miri test
```