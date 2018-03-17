# RustNotes

## How to compose functions in Rust (nightly)
from https://stackoverflow.com/questions/45786955/how-to-compose-functions-in-rust

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


