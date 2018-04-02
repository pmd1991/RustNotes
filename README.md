# Rust Notes

I am a beginner. These are some notes to help me learn Rust. I do not know if the code is any good.

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

attempt at function int parameter and return type nightly example:

```rust
#![feature(conservative_impl_trait)]

use std::collections::HashMap;

fn memoize<T>(f: T) -> impl FnMut(i32) -> i32
where
    T: Fn(i32) -> i32,
{
    let mut cache = HashMap::new();

    move |n| *cache.entry(n).or_insert_with(|| f(n))
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
```

stable...

```rust
use std::collections::HashMap;

fn memoize<'a, T>(f: T) -> Box<'a + FnMut(i32) -> i32>
where
    T: 'a + Fn(i32) -> i32,
{
    let mut cache = HashMap::new();

    Box::new(move |n| *cache.entry(n).or_insert_with(|| f(n)))
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

```

## Kleisli exercise

```rust
fn safe_root(num: f64) -> Result<f64, &'static str> {
    let result = num.sqrt();

    if result.is_nan() {
        Err("Square root of negative")
    } else {
        Ok(result)
    }
}

fn safe_reciprocal(num: f64) -> Result<f64, &'static str> {
    let result = num.recip();
    
    if result.is_infinite() {
        Err("Reciprocal of 0")
    } else {
        Ok(result)
    }
}

fn safe_root_reciprocal(num: f64) -> Result<f64, &'static str> {
    safe_reciprocal(num).and_then(|num| safe_root(num))
}

fn main() {
    println!("{:?}", safe_root_reciprocal(0.0));
    println!("{:?}", safe_root_reciprocal(-4.0));
    println!("{:?}", safe_root_reciprocal(4.0));
}


```

