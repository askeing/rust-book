# `const` 與`static`

Rust 有個方法可以利用 `const` 關鍵字宣告常數 (constant)：

```rust
const N: i32 = 5;
```

不同於 [`let`][let] 綁定, 你必須註釋一個 `const` 的型別。

[let]: variable-bindings.html

常數在整個程式的生命週期都活著。精確來說，Rust 的常數在記憶體中並沒有固定的地址。這是因為他們實際上在每個被使用的位置都被行內代換 (inlined) 了。因此，指到同一個常數的參考 (reference) 並不保證會指到同一個記憶體位置。

# `static`

Rust 提供了一個類似於全域變數 (global variable) 的機制稱為靜態 (static)。它們類似於常數，但是靜態物件在使用時並沒有被行內代換。這表示每個值只有一個實體，而且它在記憶體中有固定地址。

以下是一個範例：

```rust
static N: i32 = 5;
```

不同於 [`let`][let] 綁定, 你必須註釋 `static` 的型別。

靜態物件在整個程式的生命週期都活著，所以一個儲存在常數 (譯註：原作者疑似是將 static 誤植為 constant) 內的參照都必須有靜態生命週期 ([`'static` lifetime][lifetimes])：

```rust
static NAME: &'static str = "Steve";
```

[lifetimes]: lifetimes.html

## 可變性

你可以使用 `mut` 關鍵字來引入可變性：

```rust
static mut N: i32 = 5;
```

因為 N 成為可變的，當一個執行緒在讀取 N 的時候，可能有令一個執行緒正在寫入，造成記憶體的不安全。因此不論讀取或寫入一個 `static mut` 都是不安全的 ([`unsafe`][unsafe])，必須要把它們放在 `unsafe` 區塊中。

```rust
# static mut N: i32 = 5;

unsafe {
    N += 1;

    println!("N: {}", N);
}
```

[unsafe]: unsafe.html

此外，任何存在 `static` 的型別都必須是 `Sync`, 而且不能實作 [`Drop`][drop]。

[drop]: drop.html

# 初始化

`const` 與 `static` 都需要被指定一個值。他們只能被指定一個常數值。換句話說，你不能把一個函數的回傳值或類似複雜的值指定給它們，也不能在程式執行時指定。

# 我應該使用哪一個？

大多數時候，如果你可以在兩者之中選擇，請選擇 `const`。您很少會需要讓您的常數有固定的記憶體地址，因此藉由使用常數可以讓編譯器進行最佳化，例如讓常數不只在你的 crate 中傳遞，還能傳遞到下游的 crate。

> *commit 9eda98a*
