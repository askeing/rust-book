% 常數與靜態變數

在Rust語言中定義常數可以使用 `const` 關鍵字：

```rust
const N: i32 = 5;
```

你必須明確指定一個 `const` 的型別，這和使用 [`let`][let] 關鍵字進行綁定並不相同。

[let]: variable-bindings.html

常量作用於整個程式的生命週期。實際上，在Rust語言中常量在在內存中並沒有確定的地址，
它們會被內聯到所有被使用的地方。因此對於同一個常量的引用並不能確保您引用的是同一
個內存地址內的數據。

# 靜態變數

在Rust語言中，“全域變數”是以靜態變數的形式體現的。靜態變數與常數是類似的，只不過在靜態
變數被使用時不發生內聯。換句話說，每一個靜態變數都只有一個實體，並且位於內存中唯一確定
的位置。

這裡有一道例題：

```rust
static N: i32 = 5;
```

你必須明確指定一個`static`的型別，這和使用[`let`]關鍵字進行綁定並不相同。

靜態變數作用於整個程式的生命週期，所以任何儲存在常量中的引用都有[靜態變數生命週期][lifetimes]:

```rust
static NAME: &'static str = "Steve";
```

[lifetimes]: lifetimes.html

## 可變性

當您使用 `mut` 關鍵字的時候可以引入可變性:

```rust
static mut N: i32 = 5;
```

正因為它這種可變性導致一個執行緒正在修改變數 `N` 的時候，可能有另一個線程正在讀取
它。這樣一來，內存就處於不安全的狀態。因此無論是修改一個`static mut`，還是讀取
一個`static mut`都是不安全的, 所以它必須在 [`unsafe`][unsafe]區塊中才能操作:

```rust
# static mut N: i32 = 5;

unsafe {
    N += 1;

    println!("N: {}", N);
}
```

[unsafe]: unsafe.html

更進一步的說，任何儲存在 `static` 中的型別都必須實現 `Sync`, 而且不能實現[丟棄][drop]。

[drop]: drop.html

# 初始化

無論 `const` 還是 `static` 都需要被賦予一個值，且只能被賦予常數表達式的值。 換句話說，您不能使用一個函數的返回值或者任何相似的復合值對它賦值，也不能在
程式運行的過程中賦值。

# 我應該選擇使用哪一種構造?

當您無所謂選擇哪個的絕大多數時候就選擇 `const` 。 只有極少數的情況下您需要關心
常數在記憶體中的位址，而且使用 `const`允許您在自己的crate和衍生的crate中像常數擴展那樣優化它。

>  譯者注：Rust語言中的常量相當於C語言中的#define (參考：[rust-lang/rust#36039](https://github.com/rust-lang/rust/issues/36039) )


> crate這個詞我們還沒有決定到底要不要翻譯，一些漢譯本中把它翻譯為箱。這看起來是好的
，可是漢語中的箱和Rust中說的crate又不完全是相同的意思。不過像Java語言中的package翻譯成包已
經能夠被接受了。Rust語言中的crate和Java語言中的package是類似的。


> *commit 9eda98a*
