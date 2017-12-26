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

Within a given `mod`, you can declare sub-`mod`s. We can refer to sub-modules
with double-colon (`::`) notation: our four nested modules are
`english::greetings`, `english::farewells`, `japanese::greetings`, and
`japanese::farewells`. Because these sub-modules are namespaced under their
parent module, the names don’t conflict: `english::greetings` and
`japanese::greetings` are distinct, even though their names are both
`greetings`.

Because this crate does not have a `main()` function, and is called `lib.rs`,
Cargo will build this crate as a library:

```bash
$ cargo build
   Compiling phrases v0.0.1 (file:///home/you/projects/phrases)
$ ls target/debug
build  deps  examples  libphrases-a7448e02a0468eaa.rlib  native
```

`libphrases-hash.rlib` is the compiled crate. Before we see how to use this
crate from another crate, let’s break it up into multiple files.

# Multiple file crates

If each crate were just one file, these files would get very large. It’s often
easier to split up crates into multiple files, and Rust supports this in two
ways.

Instead of declaring a module like this:

```rust,ignore
mod english {
    // contents of our module go here
}
```

We can instead declare our module like this:

```rust,ignore
mod english;
```

If we do that, Rust will expect to find either a `english.rs` file, or a
`english/mod.rs` file with the contents of our module.

Note that in these files, you don’t need to re-declare the module: that’s
already been done with the initial `mod` declaration.

Using these two techniques, we can break up our crate into two directories and
seven files:

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

`src/lib.rs` is our crate root, and looks like this:

```rust,ignore
mod english;
mod japanese;
```

These two declarations tell Rust to look for either `src/english.rs` and
`src/japanese.rs`, or `src/english/mod.rs` and `src/japanese/mod.rs`, depending
on our preference. In this case, because our modules have sub-modules, we’ve
chosen the second. Both `src/english/mod.rs` and `src/japanese/mod.rs` look
like this:

```rust,ignore
mod greetings;
mod farewells;
```

Again, these declarations tell Rust to look for either
`src/english/greetings.rs` and `src/japanese/greetings.rs` or
`src/english/farewells/mod.rs` and `src/japanese/farewells/mod.rs`. Because
these sub-modules don’t have their own sub-modules, we’ve chosen to make them
`src/english/greetings.rs` and `src/japanese/farewells.rs`. Whew!

The contents of `src/english/greetings.rs` and `src/japanese/farewells.rs` are
both empty at the moment. Let’s add some functions.

Put this in `src/english/greetings.rs`:

```rust
fn hello() -> String {
    "Hello!".to_string()
}
```

Put this in `src/english/farewells.rs`:

```rust
fn goodbye() -> String {
    "Goodbye.".to_string()
}
```

Put this in `src/japanese/greetings.rs`:

```rust
fn hello() -> String {
    "こんにちは".to_string()
}
```

Of course, you can copy and paste this from this web page, or type
something else. It’s not important that you actually put ‘konnichiwa’ to learn
about the module system.

Put this in `src/japanese/farewells.rs`:

```rust
fn goodbye() -> String {
    "さようなら".to_string()
}
```

(This is ‘Sayōnara’, if you’re curious.)

Now that we have some functionality in our crate, let’s try to use it from
another crate.

# Importing External Crates

We have a library crate. Let’s make an executable crate that imports and uses
our library.

Make a `src/main.rs` and put this in it (it won’t quite compile yet):

```rust,ignore
extern crate phrases;

fn main() {
    println!("Hello in English: {}", phrases::english::greetings::hello());
    println!("Goodbye in English: {}", phrases::english::farewells::goodbye());

    println!("Hello in Japanese: {}", phrases::japanese::greetings::hello());
    println!("Goodbye in Japanese: {}", phrases::japanese::farewells::goodbye());
}
```

The `extern crate` declaration tells Rust that we need to compile and link to
the `phrases` crate. We can then use `phrases`’ modules in this one. As we
mentioned earlier, you can use double colons to refer to sub-modules and the
functions inside of them.

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
