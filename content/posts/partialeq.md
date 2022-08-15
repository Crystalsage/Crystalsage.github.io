+++
title = "Partial ordering and Rust"
date=2022-02-28
description = "Partial ordering in mathematics and how to utilize it in Rust."
showFullContent = false
readingTime = false
+++

> Without order, there is chaos.

## Introduction

Wikipedia defines a partial order as,

>    A reflexive, weak, or non-strict partial order is a homogeneous relation ≤ on a set P that is reflexive, antisymmetric, and transitive.

In other words, we have that in a set $P$ , $\forall a,b,c \in P$ , we must have,

1. **Reflexivity**: a≤a , i.e. every element is related to itself.
2. **Antisymmetry**: a≤b and b≤a then a=b , i.e. no two distinct elements precede each other.
3. **Transitivity**: If a≤b and b≤c then a≤c

Wow! Quite a definition. But what does it actually mean…?

## The scenario

Suppose that you want to buy a new fancy umbrella for you and your friend. These umbrellas differ in size and you would like the one which can fit both you and your friend.

Instantly, you have a perspective from which you start seeing things. You apply a filter according to your use cases, and judge the objects according to them. In your case, the filter is **size**.

Defining an _order_ on a set is the same thing!

Defining an order gives us a sense of comparison between objects of that particular set. In the example stated before, the set is the entire category of umbrellas and the objects are the umbrellas.

## Formalizing the idea
Continuing with the umbrella mould, let us state the same thing in a proper manner, so that Wikipedia’s definition fits our example, at least intuitively.

Let us define an order on the set of umbrellas. The set is $U$ and the objects of this set are umbrellas.

$$U=\{u_1,u_2,u_3,\cdots,u_n\}$$

Let’s define an operator ≤
on set U which defines a partial order. This operator gives us a comparison between sizes of umbrellas. The operator ≤

must satisfy the 3 rules stated in the opening section. Let’s see if we can explain them according to our scenario. (Note: A partially ordered set is also known as a poset.)

1. Reflexivity: An umbrella ua is related to itself. In terms of the size, this would mean that the umbrella is similarly sized to say, an exact copy of ua .

2. Antisymmetry: If two umbrellas are approximately equally sized, it’s confusing at a cursory glance whether if one really is smaller than the other. The situation can then be thought of as a scenario where both ua≤ub and ub≤ua are ’true’. So ua=ub must be true.

3. Transitivity: This is pretty obvious. Among a small, a medium and a large umbrella, there’s a clear hierarchy. If ua≤ub and ub≤uc then ua≤uc.

## Ordering in Rust

Ordering a data structure in Rust allows us to do abstract comparisons between objects of the same data structure. Let’s continue the umbrella saga.

Suppose we create a struct for describing an umbrella. The size field is of the Size type, which is user-defined.

```rust
enum Size{
  Small,
  Medium,
  Large,
}

struct Umbrella{
  size : Size,
  cost : u16,
}
```

Now, we would like to say that an umbrella is ‘better’ (for us) if the size is maximum. As a side effect, we would also love that the cost gets minimized. In other words, we’d like to define a clear notion of comparison.

In Rust, we can implement some traits, described below.

- `PartialOrd` : This trait implements the partial_cmp method where we describe what fields will be compared.
- `PartialEq` : This defines equality between objects.

Let’s do this.

```rust
impl PartialOrd for Umbrella {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
        self.size.partial_cmp(&other.size)
    }
}

impl PartialEq for Umbrella {
    fn eq(&self, other: &Self) -> bool {
        self.size == other.size
    }
}
```

but we run into a problem as soon as we run this.

```
error[E0599]: the method `partial_cmp` exists for enum `Size`, but its 
trait bounds were not satisfied
```

Oops! We forgot a really important thing here. We never defined a partial order on the `Size` enum in the first place, so the compiler doesn’t know how to compare its members. Let’s fix this.

We can define the traits described above for the `enum` as well. But, we’ll take a shortcut this time. We can use the `#derive` attribute to tell the compiler to infer the traits onto the data type without manually defining them.

```rust
// Small < Medium < Large is inferred due to discriminants.
// Implement `PartialOrd` and `PartialEq`
#[derive(PartialOrd, PartialEq)]
enum Size{
  Small,
  Medium,
  Large,
}
```

Note that variants ordered in this way are ordered by their discriminants, which are like indices. So an enum member at the top has a lower index than the one below it and is considered lesser. (See here)

Neat! Let’s try running this now.

```rust
fn main(){
  let u1 = Umbrella{ size: Size::Small, cost: 20};
  let u2 = Umbrella{ size: Size::Medium, cost: 20};
  let u3 = Umbrella{ size: Size::Medium, cost: 30};
  let u4 = Umbrella{ size: Size::Large, cost: 40};

  assert!(u1 < u2);
  assert!(u1 < u3);
  assert!(u1 < u2 && u2 < u3);
  assert!(u4 == u3);

  println!("Passed all checks!");
}
```

```bash
$ cargo run
Finished dev [unoptimized + debuginfo] target(s) in 0.00s
Running `target/debug/partialeq`
```

Passed all checks!

Awesome! We just implemented a partial order on a custom data type. The entire code can be found here: Code


## Exercise to the reader
The reader is invited to solve the following problems.

- What happens if the size of an umbrella is unknown? How will you rewrite the code to handle the situation?

- You’ll find that u2 is actually better than u3 because you can get a medium sized umbrella at lesser price. Recall that we never used the cost field in our code. Rewrite the code so we can find larger umbrellas at equal or lesser price.

## Further reading

Ordering is a fascinating topic. Many mathematicians and enthusiasts infinitely better than myself have spent much ink and chalk explaining what it means.

For starters, refer to the Order relations (Section 4.4) from Learning To Reason, where the authors offer a brief but completely sufficient explanation of ordering.


$$\blacksquare\blacksquare\blacksquare$$
