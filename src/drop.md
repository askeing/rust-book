# Drop
我們已經討論過 Trait 了。現在讓我們來談一個 Rust standard library 的特殊 Trait，[`Drop`][drop]。`Drop` 為程式執行時，如果值超出了有效範圍， 提供了一個方法。 例如︰

[drop]: ../std/ops/trait.Drop.html

```rust=
struct HasDrop;

impl Drop for HasDrop {
    fn drop(&mut self) {
        println!("Dropping!");
    }
}

fn main() {
    let x = HasDrop;

    // 做些事

} // x 超出了有效範圍

```

當程式跑到 `main()` 的結尾，`x` 就超出了有效範圍，此時程式會執行 Drop。`Drop` 只有一個方法，這方法也叫 `drop()`。 drop 會接收一個可變的 `self` 參照。

就這樣! `Drop` 的機制非常簡單，但是有些幽微之處。例如，值會依照他們被宣告的相反順序被丟棄 (dropped)。以下是範例︰

```rust=
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

這會輸出︰


```text
BOOM times 100!!!
BOOM times 1!!!
```

因為 tnt 變數是比較晚被宣告的，所以 tnt 會比 firecraker 變數提早離開有效範圍。先進後出 (Last in, first out)。

所以 `Drop` 有什樣的好處呢？一般來說，`Drop` 被用來清理與一個 struct 有關的任何資源。例如，[`Arc<T>` type][arc] 是一個參照計數型別，當 Drop 被呼叫時，它會減少參照的數目。當參照總數等於 0 時，它便會把底下的值清除。

[arc]: ../std/sync/struct.Arc.html

> *commit 024aa9a*
