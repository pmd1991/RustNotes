# Rust Notes

## How to compose functions in Rust (nightly)
from https://stackoverflow.com/questions/45786955/how-to-compose-functions-in-rust

do not know macros currently...

```rust
#![feature(conservative_impl_trait)]

macro_rules! compose {
    ( $last:expr ) => { $last };
    ( $head:expr, $($tail:expr), +) => {
        compose_two($head, compose!($($tail),+))
    };
}

fn compose_two<A, B, C, G, F>(f: F, g: G) -> impl Fn(A) -> C
where
    F: Fn(A) -> B,
    G: Fn(B) -> C,
{
    move |x| g(f(x))
}

fn main() {
    let add = |x| x + 2;
    let multiply = |x| x * 2;
    let divide = |x| x / 2;
    let intermediate = compose!(add, multiply, divide);

    let subtract = |x| x - 2;
    let finally = compose!(intermediate, subtract);

    println!("Result is {}", finally(10));
}
```

## identity function (from stack overflow)

```rust
fn identity<T>(f : T) -> T { f }
```

## memoization 

cached crate is available

attempt at int parameter example:

use std::collections::HashMap;
use std::collections::hash_map::Entry;

fn memoize<'a, T>(f: T) -> Box<'a + FnMut(i32) -> i32>
where
    T: 'a + Fn(i32) -> i32,
{
    let mut cache = HashMap::new();

    Box::new(move |n| match cache.entry(n) {
        Entry::Occupied(entry) => { *entry.get() },
        Entry::Vacant(entry) => { *entry.insert(f(n)) },
    })
}

fn fake_slow(i: i32) -> i32 {
    println!("slow operation");
    i
}

fn main() {
    let mut f = memoize(fake_slow);

    println!("{}", f(4));
    println!("{}", f(4));
    println!("{}", f(2));
}

