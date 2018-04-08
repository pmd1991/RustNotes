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

For more information, it looks like there is a crate with a Kleisli struct.

```rust
fn safe_root(num: f64) -> (f64, &'static str) {
    let num = num.sqrt();
    
    if num.is_finite() {
        (num, "")
    } else {
        (std::f64::NAN, "Error taking square root. ")
    }
}

fn safe_reciprocal(num: f64) -> (f64, &'static str) {
    let num = num.recip();
    
    if num.is_finite() {
        (num, "")
    } else {
        (std::f64::NAN, "Reciprocal error. ")
    }
}

fn safe_root_reciprocal(num: f64) -> (f64, String) {
    let result1 = safe_reciprocal(num);
    let result2 = safe_root(result1.0);

    (result2.0, format!("{}{}", result1.1, result2.1))
}

fn main() {
    println!("1. compose 4 {:?}", safe_root_reciprocal(4.0));
    println!("2. compose 0 {:?}", safe_root_reciprocal(0.0));
    println!("3. compose -4 {:?}", safe_root_reciprocal(-4.0));
    println!("4. compose inf {:?}", safe_root_reciprocal(std::f64::INFINITY));
    println!("5. compose neg inf {:?}", safe_root_reciprocal(std::f64::NEG_INFINITY));
    println!("6. compose nan {:?}", safe_root_reciprocal(std::f64::NAN));
    println!("7. compose min positive: {:?}", safe_root_reciprocal(std::f64::MIN_POSITIVE));
    println!("8. compose max: {:?}", safe_root_reciprocal(std::f64::MAX));
    println!("1a. root epsilon: {:?}", safe_root(std::f64::EPSILON));
    println!("2a. root inf {:?}", safe_root(std::f64::INFINITY));
    println!("3a. root neg inf {:?}", safe_root(std::f64::NEG_INFINITY));
    println!("4a. root nan {:?}", safe_root(std::f64::NAN));
    println!("5a. root min positive: {:?}", safe_root(std::f64::MIN_POSITIVE));
    println!("6a. root max: {:?}", safe_root(std::f64::MAX));
    println!("7a. root epsilon: {:?}", safe_root(std::f64::EPSILON));
    println!("8a. root zero {:?}", safe_root(0.0));
    println!("1b. reciprocal inf {:?}", safe_reciprocal(std::f64::INFINITY));
    println!("2b. reciprocal neg inf {:?}", safe_reciprocal(std::f64::NEG_INFINITY));
    println!("3b. reciprocal nan {:?}", safe_reciprocal(std::f64::NAN));
    println!("4b. reciprocal min positive: {:?}", safe_reciprocal(std::f64::MIN_POSITIVE));
    println!("5b. reciprocal max: {:?}", safe_reciprocal(std::f64::MAX));
    println!("6b. reciprocal epsilon: {:?}", safe_reciprocal(std::f64::EPSILON));
    println!("7b. reciprocal 0 {:?}", safe_reciprocal(0.0));
}
```

