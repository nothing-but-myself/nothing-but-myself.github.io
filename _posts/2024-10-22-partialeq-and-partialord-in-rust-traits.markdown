---
layout: post
title: Rust Trait 中的 PartialEq 和 PartialOrd
---
我们都知道 rust 中有很多可以派生的 trait，在属性上增加 _#[derive(...)]_ 的相关标记即可默认为结构体或枚举生成对应 trait 的默认实现，这里我们讨论两种 tarit 实现，一种是等值比较，另一种是次序比较，看看他们有什么奥妙。

# 等值比较

PartialEq 和 Eq 都用作等值比较，但不同在于 PartialEq 强调部分的、不完整的等同性，这也正是其名字中 Partial 的由来。

PartialEq 这个 trait 默认实现了 eq 方法，而由于全等的肯定是部分全等的，所以无论是 PartialEq 还是 Eq 的类型都可以使用 eq 这个方法来做两相比较。

全等关系需要满足三个条件
- 自反性，对于任何 a，那么 a == a 应该为真。
- 对称性，对于任何 a 满足 a == b，那么 b == a 应该为真。
- 传递性，对于任何 a 满足 a == b 且 b == c，那么 a == c 应该为真。

然而对与 rust 浮点数中 _NaN_ 这个特殊值来说，不满足自反性，故而浮点类型只能是一个部分全等的类型。我们可以具体看下

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

```rust
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

```rust
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

PartialOrd 提供了 partial_cmp 的方法。有了上面的经验，我们也先来看一下函数签名

```rust
fn partial_cmp(&self, other: &Rhs) -> Option<Ordering>;
```

由此可见，它需要和一个引用比较，且返回一个 Option 的 Ordering。我们盲猜 Ordering 是一个枚举，无非产生集中结果，比大、比小、相等。 但为什么是 Option 呢？

原来有些情况是无法产生次序的，此时就需要返回 None。除了能比（正常执行），不能比（异常退出），还有无法比较的情形。这里又得要提及浮点数的 NaN 了，它们之间是无法比较的。我们实际看几个例子

```rust
fn main() {
    let gt = 1.partial_cmp(&0);
    println!("{:?}", gt);
    
    let lt = 0.partial_cmp(&1);
    println!("{:?}", lt);
    
    let eq = 0.partial_cmp(&0);
    println!("{:?}", eq);
    
    let none = f32::NAN.partial_cmp(&f32::NAN);
    println!("{:?}", none);
}
```

Standard Output

```rust
Some(Greater)
Some(Less)
Some(Equal)
None
```

结构体的比较是按照字段的次序依次比较的，看例子之前，必须要严格注意实现 PartialOrd 必须实现 PartialEq

```rust
#[derive(PartialEq, PartialOrd)]
struct UnitOrder(u8,u8,u8);

fn main() {
    let u1 = UnitOrder(1,2,3);
    let u2 = UnitOrder(1,3,2);
    let lt = u1.partial_cmp(&u2);
    println!("{:?}", lt);
}
```

Standard Output

```rust
Some(Less)
```

枚举比较有意思，它们在实现了这个 trait 之后，它们之间只能相互比较自己的变体，且位置较前声明的变体，总是大于较晚声明的变体。

```rust
#[derive(PartialEq, PartialOrd)]
enum EnumOrder{
    A,
    B,
}

fn main() {
    let a = EnumOrder::A;
    let b = EnumOrder::B;
    let lt = a.partial_cmp(&b);
    println!("{:?}", lt);
}
```

Standard Output

```rust
Some(Less)
```

此外，还有一个 Ord 的 trait，这个 trait 是 Eq 和 PartialOrd 的结合，我们知道有 Eq 那么必然有 PartialEq， 也就是必然能判别是否相等。与此同时，再混合上 ParitalOrd，正好弥补了存在 None 的可能性，因而 Ord 总能捋出个顺序来，由此可知它返回的肯定不再是 Option 了，实际看下它的方法（提供了 cmp 方法）签名，返回的是个 Ordering 枚举

```rust
fn cmp(&self, other: &Self) -> Ordering;
```

写个例子瞅瞅

```rust
#[derive(PartialEq, Eq, PartialOrd, Ord)]
enum EnumOrder {
    A
}

fn main() {
    let a1 = EnumOrder::A;
    let a2 = EnumOrder::A;
    let eq = a1.cmp(&a2);
    println!("{:?}", eq);
}
```

Standard Output

```rust
Equal
```

从 Ord 派成的结构体和枚举的行为和 PartialOrd 一致，有兴趣可以多写写看。

# 尾巴

这次仅仅介绍了等值和次序比较，涉及到了以下两组traits：
- PartialEq & Eq
- PartialOrd & Ord

写例子的时候，联想到 Python 中的一组魔法方法：`__lt__`、`__gt__`、`__le__`、`__ge__`、`__eq__` 和 `__ne__`，不得不惊讶于编程的相通之处，感叹 trait 也可能是种 rust 的魔法吧。
