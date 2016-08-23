% Drop

Now that we’ve discussed traits, let’s talk about a particular trait provided
by the Rust standard library, [`Drop`][drop]. The `Drop` trait provides a way
to run some code when a value goes out of scope. For example:

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

When `x` goes out of scope at the end of `main()`, the code for `Drop` will
run. `Drop` has one method, which is also called `drop()`. It takes a mutable
reference to `self`.

That’s it! The mechanics of `Drop` are very simple, but there are some
subtleties. For example, values are dropped in the opposite order they are
declared. Here’s another example:

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

那麼 `Drop` 有什麼好處呢？通常情況下 `Drop` 被用來清理和 `struct` 關聯得到資源. 譬如說[`Arc<T>` 型別][arc]是一個引用計數型別。當 `Drop` 被調用的時候，它就會減少引用計數。此外，如果引用的總數為零，底層的值將會被擦除。

[arc]: ../std/sync/struct.Arc.html


> *commit 024aa9a*
