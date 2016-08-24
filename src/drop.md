% 丟棄

基於我們已經討論過特征了，現在讓我們一起來了解由Rust語言標準庫提供的特殊
的特征—— [`Drop`(丟棄)][drop].  丟棄的特征是由一個值離開作用域時觸發的方
法，譬如說：

[drop]: ../std/ops/trait.Drop.html

```rust
struct HasDrop;

impl Drop for HasDrop {
    fn drop(&mut self) {
        println!("Dropping!");
    }
}

fn main() {
    let x = HasDrop;

    // do stuff

} // x goes out of scope here
```

當變數 `x` 離開它的作用域 `main()`的底部的時候, 丟棄 `Drop`的代碼
就被觸發了。 丟棄 `Drop`有一個方法也被寫作 `drop()`。它用來回去一個自身`self`的可變引用。

嗚呼！ 丟棄 `Drop` 的運行原理非常簡單。只是還有一些細節問題：譬
如說值會被它們聲明的相反順序觸發丟棄的過程，譬如說：

```rust
struct Firework {
    strength: i32,
}

impl Drop for Firework {
    fn drop(&mut self) {
        println!("BOOM times {}!!!", self.strength);
    }
}

fn main() {
    let firecracker = Firework { strength: 1 };
    let tnt = Firework { strength: 100 };
}
```

輸出結果：

```text
BOOM times 100!!!
BOOM times 1!!!
```

由於TNT在firecracker之後被聲明，所以TNT在firecracker之前離開作用域。

那麼 `Drop`  有什麼好處呢？通常情況下 `Drop` 被用來清理和 `struct`關聯得到資源。
譬如說 [`Arc<T>` 型別][arc] 是一個引用計數型別。當`Drop`被調用的時候，它就會減
少引用計數。此外，如果引用的總數為零，底層的值將會被擦除。


[arc]: ../std/sync/struct.Arc.html


> *commit 024aa9a*
