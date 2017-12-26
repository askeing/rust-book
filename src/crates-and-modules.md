# Crates 和 Modules

當一個專案開始變大的時候，將它切小再把他們組裝在一起會被認為是個好的軟體工程作法。
有個定義得很好的介面也很重要，這麼做的話你就可以將某些 functionality 區分成公開及私有的。
為了滿足上述的需求，Rust 便提供了模組系統 (module system)。

# 基本用詞：Crates 以及 Modules

Rust 有兩個獨立的詞和模組系統相關： `crate` 以及 `module`。crate 跟其他語言中的函式庫 (library) 和套件包 (package) 是同義詞。 
因此我們用 Cargo 作為 rust 的套件管理工具：你會用貨櫃 (cargo) 把你的木箱 (crate) 運送到其他人那裡。 
Crates 可以是可執行的，也可以是函式庫，這取決於專案。

每個 crate 都有隱含的 *root module*，用來包含該 crate 的程式。
接著，你可以在該 root module 下定義一個子模組樹 (sub-modules tree)。
模組讓你可以將你的程式碼依照 crate 分區。

舉個例子，我們建立一個 *句子 (phrases)* crate，這可以給我們不同語言的幾個句子。
為了讓事情簡單化，我們先使用 `greetings` 和 `farewells` 這兩種句子，並使用英文及日文這兩種語言來舉例。
我們會有下面的 module layout：


```text
                                    +-----------+
                                +---| greetings |
                                |   +-----------+
                  +---------+   |
              +---| english |---+
              |   +---------+   |   +-----------+
              |                 +---| farewells |
+---------+   |                     +-----------+
| phrases |---+
+---------+   |                     +-----------+
              |                 +---| greetings |
              |   +----------+  |   +-----------+
              +---| japanese |--+
                  +----------+  |
                                |   +-----------+
                                +---| farewells |
                                    +-----------+
```

在這個範例中，`phrases` 就是我們的 crate 的名字。剩下的東西就是模組。
如你所見，他們形成了一棵樹，從 crate 作為 *根 (root)* 開始分岔出去。

現在我們有個計畫，讓我們用程式碼定義這些模組吧。
我們可以用 Cargo 來產生個新的 crate：

```bash
$ cargo new phrases
$ cd phrases
```

如果你記得的話，這會替我們產生個新的專案：

```bash
$ tree .
.
├── Cargo.toml
└── src
    └── lib.rs

1 directory, 2 files
```

`src/lib.rs` 是我們 crate 的 root，對應到我們的圖中的 `phrases`。

# 定義模組

要定義我們的每個模組，我們會使用 `mod` 這個關鍵字。讓我們建立我們的
`src/lib.rs`：

```rust
mod english {
    mod greetings {
    }

    mod farewells {
    }
}

mod japanese {
    mod greetings {
    }

    mod farewells {
    }
}
```

在 `mod` 之後，你會給定模組的名字。模組名的規定會跟其他 Rust identifiers 相同： `lower_snake_case`。
而模組的內容則用 curly braces (`{}`) 包住。

在給定的 `mod` 中，你會宣告子 `mod`。我們會將用雙冒號參照到子模組：
我們的四個巢狀模組分別是 `english::greetings`、`english::farewells`、`japanese::greetings`，以及 `japanese::farewells`。
因為這些子模組都隸屬於 (namespaced) 他們的父模組之下，名字便不會衝突到：
即使 `english::greetings` 和 `japanese::greetings` 的名字都叫 `greetings`，但他們仍互相獨立。
 
因為這個 crate 沒有 `main()` function，且被命名為 `lib.rs`，
Cargo 會將它建成函式庫：

```bash
$ cargo build
   Compiling phrases v0.0.1 (file:///home/you/projects/phrases)
$ ls target/debug
build  deps  examples  libphrases-a7448e02a0468eaa.rlib  native
```

`libphrases-hash.rlib` 是個被編譯出來的 crate。在我們看要怎麼從別的 crate 來用這個
crate 之前，讓我們把他切成幾個檔案。

# 多檔 crates

如果每個 crate 都只是一個檔案的話，那那個檔案一定會變得非常巨大。
所以我們通常會將 crate 切成多個檔案，而 rust 支援兩種方式。


不像這樣宣告模組：

```rust,ignore
mod english {
    // contents of our module go here
}
```

我們也可以像這樣軒高我們的模組：

```rust,ignore
mod english;
```

如果我們這麼做的話，Rust 會預期專案中會有個叫做 `english.rs` 或是 `english/mod.rs` 的檔案。

要注意的是，在這些檔案中，你不用再次宣告你的模組：我們已經在初始化 `mod` 時宣告過了。

使用這兩中方法的話，我們可以將我們的 crate 切成兩個目錄、七個檔案：

```bash
$ tree .
.
├── Cargo.lock
├── Cargo.toml
├── src
│   ├── english
│   │   ├── farewells.rs
│   │   ├── greetings.rs
│   │   └── mod.rs
│   ├── japanese
│   │   ├── farewells.rs
│   │   ├── greetings.rs
│   │   └── mod.rs
│   └── lib.rs
└── target
    └── debug
        ├── build
        ├── deps
        ├── examples
        ├── libphrases-a7448e02a0468eaa.rlib
        └── native
```

`src/lib.rs` 是我們的 crate root，而且長得像這樣：

```rust,ignore
mod english;
mod japanese;
```

這兩個宣告會叫 Rust 去找 `src/english.rs` 和`src/japanese.rs`，
或是 `src/english/mod.rs` 和 `src/japanese/mod.rs`，取決於我們的喜好。

在這個例子中，因為我們的這兩個模組各自有子模組，所以我們選擇第二種方式。

`src/english/mod.rs` 和 `src/japanese/mod.rs` 都會長得像這樣：

```rust,ignore
mod greetings;
mod farewells;
```

再一次地，這些宣告會叫 Rust 去找 `src/english/greetings.rs` 和 `src/japanese/greetings.rs`，
或是 `src/english/farewells/mod.rs` 和 `src/japanese/farewells/mod.rs`。
因為這兩個組子模組都沒有屬於他們的子模組了，我們可以直接放在
`src/english/greetings.rs` 和 `src/japanese/farewells.rs`。呼～～！

`src/english/greetings.rs` 和 `src/japanese/farewells.rs` 的內容在此時都會是空的。
我們來加一些函式吧：

把這個放在 `src/english/greetings.rs` 中：

```rust
fn hello() -> String {
    "Hello!".to_string()
}
```

把這個放在 `src/english/farewells.rs` 中：

```rust
fn goodbye() -> String {
    "Goodbye.".to_string()
}
```

把這個放在 `src/japanese/greetings.rs` 中：

```rust
fn hello() -> String {
    "こんにちは".to_string()
}
```

當然，你可以從網頁上複製過去，或是打一些什麼別的。
你是否真的把 ‘konnichiwa’ 放進檔案中沒很重要，
有沒有這麼做你都還是可以學會模組系統。

把這個放在 `src/japanese/farewells.rs` 中：

```rust
fn goodbye() -> String {
    "さようなら".to_string()
}
```

(如果你好奇的話，這個叫做 ‘Sayōnara’。)

現在我們的 crate 有一些函式了，讓我們試著從另一個 crate 使用它吧。

# 引入外部 Crates

現在我們有了個函式庫 crate，讓我們引用他來建立一個可執行的 crate。

建立一個 `src/main.rs` 並把下面這些程式碼放進去 (先不用編譯)：

```rust,ignore
extern crate phrases;

fn main() {
    println!("Hello in English: {}", phrases::english::greetings::hello());
    println!("Goodbye in English: {}", phrases::english::farewells::goodbye());

    println!("Hello in Japanese: {}", phrases::japanese::greetings::hello());
    println!("Goodbye in Japanese: {}", phrases::japanese::farewells::goodbye());
}
```

`extern crate` 宣告會告訴 Rust 我們需要編譯並連結 (link) 到 `phrases` crate。
接著我們就能裡面使用我們的 `phrases` 模組。
如同我們先前有提到過的，你可以使用雙冒號來參照子模組中的 functions。

(Note: when importing a crate that has dashes in its name "like-this", which is
not a valid Rust identifier, it will be converted by changing the dashes to
underscores, so you would write `extern crate like_this;`.)

Also, Cargo assumes that `src/main.rs` is the crate root of a binary crate,
rather than a library crate. Our package now has two crates: `src/lib.rs` and
`src/main.rs`. This pattern is quite common for executable crates: most
functionality is in a library crate, and the executable crate uses that
library. This way, other programs can also use the library crate, and it’s also
a nice separation of concerns.

This doesn’t quite work yet, though. We get four errors that look similar to
this:

```bash
$ cargo build
   Compiling phrases v0.0.1 (file:///home/you/projects/phrases)
src/main.rs:4:38: 4:72 error: function `hello` is private
src/main.rs:4     println!("Hello in English: {}", phrases::english::greetings::hello());
                                                   ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
note: in expansion of format_args!
<std macros>:2:25: 2:58 note: expansion site
<std macros>:1:1: 2:62 note: in expansion of print!
<std macros>:3:1: 3:54 note: expansion site
<std macros>:1:1: 3:58 note: in expansion of println!
phrases/src/main.rs:4:5: 4:76 note: expansion site
```

By default, everything is private in Rust. Let’s talk about this in some more
depth.

# Exporting a Public Interface

Rust allows you to precisely control which aspects of your interface are
public, and so private is the default. To make things public, you use the `pub`
keyword. Let’s focus on the `english` module first, so let’s reduce our `src/main.rs`
to only this:

```rust,ignore
extern crate phrases;

fn main() {
    println!("Hello in English: {}", phrases::english::greetings::hello());
    println!("Goodbye in English: {}", phrases::english::farewells::goodbye());
}
```

In our `src/lib.rs`, let’s add `pub` to the `english` module declaration:

```rust,ignore
pub mod english;
mod japanese;
```

And in our `src/english/mod.rs`, let’s make both `pub`:

```rust,ignore
pub mod greetings;
pub mod farewells;
```

In our `src/english/greetings.rs`, let’s add `pub` to our `fn` declaration:

```rust,ignore
pub fn hello() -> String {
    "Hello!".to_string()
}
```

And also in `src/english/farewells.rs`:

```rust,ignore
pub fn goodbye() -> String {
    "Goodbye.".to_string()
}
```

Now, our crate compiles, albeit with warnings about not using the `japanese`
functions:

```bash
$ cargo run
   Compiling phrases v0.0.1 (file:///home/you/projects/phrases)
src/japanese/greetings.rs:1:1: 3:2 warning: function is never used: `hello`, #[warn(dead_code)] on by default
src/japanese/greetings.rs:1 fn hello() -> String {
src/japanese/greetings.rs:2     "こんにちは".to_string()
src/japanese/greetings.rs:3 }
src/japanese/farewells.rs:1:1: 3:2 warning: function is never used: `goodbye`, #[warn(dead_code)] on by default
src/japanese/farewells.rs:1 fn goodbye() -> String {
src/japanese/farewells.rs:2     "さようなら".to_string()
src/japanese/farewells.rs:3 }
     Running `target/debug/phrases`
Hello in English: Hello!
Goodbye in English: Goodbye.
```

`pub` also applies to `struct`s and their member fields. In keeping with Rust’s
tendency toward safety, simply making a `struct` public won't automatically
make its members public: you must mark the fields individually with `pub`.

Now that our functions are public, we can use them. Great! However, typing out
`phrases::english::greetings::hello()` is very long and repetitive. Rust has
another keyword for importing names into the current scope, so that you can
refer to them with shorter names. Let’s talk about `use`.

# Importing Modules with `use`

Rust has a `use` keyword, which allows us to import names into our local scope.
Let’s change our `src/main.rs` to look like this:

```rust,ignore
extern crate phrases;

use phrases::english::greetings;
use phrases::english::farewells;

fn main() {
    println!("Hello in English: {}", greetings::hello());
    println!("Goodbye in English: {}", farewells::goodbye());
}
```

The two `use` lines import each module into the local scope, so we can refer to
the functions by a much shorter name. By convention, when importing functions, it’s
considered best practice to import the module, rather than the function directly. In
other words, you _can_ do this:

```rust,ignore
extern crate phrases;

use phrases::english::greetings::hello;
use phrases::english::farewells::goodbye;

fn main() {
    println!("Hello in English: {}", hello());
    println!("Goodbye in English: {}", goodbye());
}
```

But it is not idiomatic. This is significantly more likely to introduce a
naming conflict. In our short program, it’s not a big deal, but as it grows, it
becomes a problem. If we have conflicting names, Rust will give a compilation
error. For example, if we made the `japanese` functions public, and tried to do
this:

```rust,ignore
extern crate phrases;

use phrases::english::greetings::hello;
use phrases::japanese::greetings::hello;

fn main() {
    println!("Hello in English: {}", hello());
    println!("Hello in Japanese: {}", hello());
}
```

Rust will give us a compile-time error:

```text
   Compiling phrases v0.0.1 (file:///home/you/projects/phrases)
src/main.rs:4:5: 4:40 error: a value named `hello` has already been imported in this module [E0252]
src/main.rs:4 use phrases::japanese::greetings::hello;
                  ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
error: aborting due to previous error
Could not compile `phrases`.
```

If we’re importing multiple names from the same module, we don’t have to type it out
twice. Instead of this:

```rust,ignore
use phrases::english::greetings;
use phrases::english::farewells;
```

We can use this shortcut:

```rust,ignore
use phrases::english::{greetings, farewells};
```

## Re-exporting with `pub use`

You don’t only use `use` to shorten identifiers. You can also use it inside of your crate
to re-export a function inside another module. This allows you to present an external
interface that may not directly map to your internal code organization.

Let’s look at an example. Modify your `src/main.rs` to read like this:

```rust,ignore
extern crate phrases;

use phrases::english::{greetings,farewells};
use phrases::japanese;

fn main() {
    println!("Hello in English: {}", greetings::hello());
    println!("Goodbye in English: {}", farewells::goodbye());

    println!("Hello in Japanese: {}", japanese::hello());
    println!("Goodbye in Japanese: {}", japanese::goodbye());
}
```

Then, modify your `src/lib.rs` to make the `japanese` mod public:

```rust,ignore
pub mod english;
pub mod japanese;
```

Next, make the two functions public, first in `src/japanese/greetings.rs`:

```rust,ignore
pub fn hello() -> String {
    "こんにちは".to_string()
}
```

And then in `src/japanese/farewells.rs`:

```rust,ignore
pub fn goodbye() -> String {
    "さようなら".to_string()
}
```

Finally, modify your `src/japanese/mod.rs` to read like this:

```rust,ignore
pub use self::greetings::hello;
pub use self::farewells::goodbye;

mod greetings;
mod farewells;
```

The `pub use` declaration brings the function into scope at this part of our
module hierarchy. Because we’ve `pub use`d this inside of our `japanese`
module, we now have a `phrases::japanese::hello()` function and a
`phrases::japanese::goodbye()` function, even though the code for them lives in
`phrases::japanese::greetings::hello()` and
`phrases::japanese::farewells::goodbye()`. Our internal organization doesn’t
define our external interface.

Here we have a `pub use` for each function we want to bring into the
`japanese` scope. We could alternatively use the wildcard syntax to include
everything from `greetings` into the current scope: `pub use self::greetings::*`.

What about the `self`? Well, by default, `use` declarations are absolute paths,
starting from your crate root. `self` makes that path relative to your current
place in the hierarchy instead. There’s one more special form of `use`: you can
`use super::` to reach one level up the tree from your current location. Some
people like to think of `self` as `.` and `super` as `..`, from many shells’
display for the current directory and the parent directory.

Outside of `use`, paths are relative: `foo::bar()` refers to a function inside
of `foo` relative to where we are. If that’s prefixed with `::`, as in
`::foo::bar()`, it refers to a different `foo`, an absolute path from your
crate root.

This will build and run:

```bash
$ cargo run
   Compiling phrases v0.0.1 (file:///home/you/projects/phrases)
     Running `target/debug/phrases`
Hello in English: Hello!
Goodbye in English: Goodbye.
Hello in Japanese: こんにちは
Goodbye in Japanese: さようなら
```

## Complex imports

Rust offers several advanced options that can add compactness and
convenience to your `extern crate` and `use` statements. Here is an example:

```rust,ignore
extern crate phrases as sayings;

use sayings::japanese::greetings as ja_greetings;
use sayings::japanese::farewells::*;
use sayings::english::{self, greetings as en_greetings, farewells as en_farewells};

fn main() {
    println!("Hello in English; {}", en_greetings::hello());
    println!("And in Japanese: {}", ja_greetings::hello());
    println!("Goodbye in English: {}", english::farewells::goodbye());
    println!("Again: {}", en_farewells::goodbye());
    println!("And in Japanese: {}", goodbye());
}
```

What's going on here?

First, both `extern crate` and `use` allow renaming the thing that is being
imported. So the crate is still called "phrases", but here we will refer
to it as "sayings". Similarly, the first `use` statement pulls in the
`japanese::greetings` module from the crate, but makes it available as
`ja_greetings` as opposed to simply `greetings`. This can help to avoid
ambiguity when importing similarly-named items from different places.

The second `use` statement uses a star glob to bring in all public symbols from
the `sayings::japanese::farewells` module. As you can see we can later refer to
the Japanese `goodbye` function with no module qualifiers. This kind of glob
should be used sparingly. It’s worth noting that it only imports the public
symbols, even if the code doing the globbing is in the same module.

The third `use` statement bears more explanation. It's using "brace expansion"
globbing to compress three `use` statements into one (this sort of syntax
may be familiar if you've written Linux shell scripts before). The
uncompressed form of this statement would be:

```rust,ignore
use sayings::english;
use sayings::english::greetings as en_greetings;
use sayings::english::farewells as en_farewells;
```

As you can see, the curly brackets compress `use` statements for several items
under the same path, and in this context `self` refers back to that path.
Note: The curly brackets cannot be nested or mixed with star globbing.


> *commit 5c61be6*
