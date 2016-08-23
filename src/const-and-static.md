% 常量與靜態量

在Rust語言中定義常量可以使用 `const` 關鍵字:

```rust
const N: i32 = 5;
```

你必須明確一個 `const` 的數據類型，這和使用 [`let`][let] 關鍵字進行綁定並不相同。

[let]: variable-bindings.html

常量作用於整個程式的生命週期。實際上，在Rust語言中常量在在內存中並沒有確定的地址，它們會被內聯到所有被使用的地方。因此對於同一個常量的引用並不能確保您引用的是同一個內存地址內的數據。

# `static`

Rust provides a ‘global variable’ sort of facility in static items. They’re
similar to constants, but static items aren’t inlined upon use. This means that
there is only one instance for each value, and it’s at a fixed location in
memory.

這裡有一道例題：

```rust
static N: i32 = 5;
```

Unlike [`let`][let] bindings, you must annotate the type of a `static`.

Statics live for the entire lifetime of a program, and therefore any
reference stored in a constant has a [`'static` lifetime][lifetimes]:

```rust
static NAME: &'static str = "Steve";
```

[lifetimes]: lifetimes.html

## Mutability

You can introduce mutability with the `mut` keyword:

```rust
static mut N: i32 = 5;
```

Because this is mutable, one thread could be updating `N` while another is
reading it, causing memory unsafety. As such both accessing and mutating a
`static mut` is [`unsafe`][unsafe], and so must be done in an `unsafe` block:

```rust
# static mut N: i32 = 5;

unsafe {
    N += 1;

    println!("N: {}", N);
}
```

[unsafe]: unsafe.html

Furthermore, any type stored in a `static` must be `Sync`, and may not have
a [`Drop`][drop] implementation.

[drop]: drop.html

# Initializing

Both `const` and `static` have requirements for giving them a value. They may
only be given a value that’s a constant expression. In other words, you cannot
use the result of a function call or anything similarly complex or at runtime.

# Which construct should I use?

Almost always, if you can choose between the two, choose `const`. It’s pretty
rare that you actually want a memory location associated with your constant,
and using a const allows for optimizations like constant propagation not only
in your crate but downstream crates.


> *commit 9eda98a*
