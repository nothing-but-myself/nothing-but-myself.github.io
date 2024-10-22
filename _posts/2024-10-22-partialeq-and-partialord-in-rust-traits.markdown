---
layout: post
title: Rust Trait 中的 PartialEq 和 PartialOrd
---
我们都知道 rust 中有很多可以派生的 trait，在属性上增加 `#[derive(...)]` 的相关标记即可默认为结构体或枚举生成对应 trait 的默认实现，这里我们讨论两种 tarit 实现，一种是等值比较，另一种是次序比较，看看他们有什么奥妙。

# 等值比较

PartialEq 和 Eq 都用作等值比较，但不同在于 PartialEq 强调部分的、不完整的等同性，这也正是其名字中 Partial 的由来。

PartialEq 这个 trait 默认实现了 eq 方法，而由于全等的肯定是部分全等的，所以无论是 PartialEq 还是 Eq 的类型都可以使用 eq 这个方法来做两相比较。

全等关系需要满足三个条件
- 自反性，对于任何 a，那么 a == a 应该为真。
- 对称性，对于任何 a 满足 a == b，那么 b == a 应该为真。
- 传递性，对于任何 a 满足 a == b 且 b == c，那么 a == c 应该为真。

然而对与 rust 浮点数中 `NaN` 这个特殊值来说，不满足自反性，故而浮点类型只能是一个部分全等的类型。我们可以具体看下

```rust
fn main() {
    let eq_false = f64::NAN == f64::NAN;
    println!("{eq_false}");
    
    let eq_false = f64::NAN.eq(&f64::NAN);
    println!("{eq_false}");
    
    let eq_true = 1.0f64 == 1.00;
    println!("{eq_true}")
}
```

Standard Output 

```shell
false
false
true
```

区别于部分全等，Eq 是一个满足自反性的真全等类型。因为 Eq Trait 没有任何方法，所以 Eq 的出现肯定是伴随 PartialEq 的，不然没法使用 eq 方法进行比较，反之则不然。我们这里自己写一个 Struct 来展示

```rust
#[derive(PartialEq, Eq)]
struct RealEqual;

#[derive(PartialEq, Eq)]
struct RealEqual2(i32);

fn main() {
    let a = RealEqual;
    let b = RealEqual;
    println!("{}", a==b);
    
    let c = RealEqual2(8);
    let d = RealEqual2(8);
    let e = RealEqual2(9);
    
    println!("{}", c.eq(&d));
    println!("{}", c.eq(&e));
}
```

Standard Output 

```shell
true
true
false
```

这里的 derive 中 Eq 应该不会单独出现，必然伴随 PartialEq。

另外我们再较真一下 eq 这个方法的签名，传入的是一个引用，返回的是一个 bool， 正如我们上面代码中所展示的那样。

```rust
fn eq(&self, other: &Rhs) -> bool;
```

# 次序比较

占坑

