---
title: What type is arr[i]? Indexing in Go vs Rust
slug: indexing-type-go-vs-rust
description: The type behind arr[i]: why Rust picks usize and Go picks int.
language: en
is_favorite: false
tags:
  - go
  - rust
published_at:
---

I write Go in my daily work and I really like the language. Recently I started learning Rust, and I'm enjoying it a lot. I think learning a new language is fun and it also makes you a better programmer in your main language.
Every time you learn something new in the new language, you catch yourself thinking: "How would I do this in Go?" or "How does Go implement this under the hood?" And with AI around, this comparison is easier and you end up learning a ton.

A side note before we start: people used to talk more about their experience learning a new language or a new tool. Now almost all the content is about AI, and honestly it gets a bit boring.
Some people ask, "Why bother learning new things when AI does it better?" I don't buy that. You don't walk up to a football fan and ask why he knows all these players or the whole World Cup history. You don't ask a movie lover why he remembers every little detail. He knows because he loves it. I love programming and I enjoy it. I don't care if AI does it better than me or whatever. I don't consider myself a great programmer, but I'm trying to be one and if AI takes my job, I'm sure I'll still be coding after I get home from the goose farm. :)
(That's how I feel right now, and I hope it doesn't change.) Sorry, I just used this blog post to leave a note for myself. Back to the story.

I was reading a book about Rust when I hit this sentence: "Rust uses `usize` for indexing." I paused. Wait, *indexing has a type?* I had never really thought about it.

## Type for Indexing

### Rust

```rust
fn main() {
    let arr: [i32; 3] = [1, 2, 3];

    let i: usize = 1;
    _ = arr[i]; // ok, the index is a usize

    let j: i32 = 1;
    // _ = arr[j];       // error: expected `usize`, found `i32`
    _ = arr[j as usize]; // ok, but you must cast
}
```

- **Index type:**
    - The index expression must be a `usize`.
    - `usize` is an **unsigned**, architecture-dependent integer whose size matches the platform's pointer size.
- **Why `usize`?**
    - An array index can never be negative, so an unsigned type makes sense.
    - An index is not a memory address; it is an offset from the start of the array.
    - When you write `arr[i]`, the compiler computes the element's address as `base_pointer + i * size_of::<T>()`.
    - So the index type must be large enough to represent the offset of any element in the largest possible allocation.
    - Since the maximum allocation size is limited by the machine's pointer size, Rust uses a pointer-sized unsigned integer (`usize`).
- **Bounds checking:** `arr[i]` is checked at runtime and **panics** if `i` is out of bounds. The compiler may remove the check when it can prove at compile time that the index is always valid. The safe, non-panicking alternative is `arr.get(i)`, which returns an `Option<&T>`.
- **Length and arithmetic:** `arr.len()` returns a `usize` (unsigned).
    - `arr.len() - 1` when the array is empty (`len == 0`):
        - **Debug build:** panics (integer overflow).
        - **Release build:** wraps to `usize::MAX`.
    - Because of this, idiomatic reverse iteration uses `(0..n).rev()` or `arr.iter().rev()` instead of counting down from `n - 1`.

> **What about `let y = 1; arr[y]`? Isn't the default `i32`?** That still compiles, and it's
> *not* an implicit cast. Rust integer literals start out untyped; `i32` is only the *fallback*
> the compiler picks when nothing constrains them. Here `arr[y]` requires `y: usize`, and that
> requirement flows backward through the expression, so the literals are inferred as `usize`
> from the start. Nothing is coerced. The moment you pin a type, like `let y: i32 = 1;`, the
> same code stops compiling, because Rust never casts between integer types implicitly.

### Go

```go
func main() {
	var (
		arr       = [3]int{1, 2, 3}
		x   int8  = 0
		y   uint8 = 0
	)
	_ = arr[1] // 1 here is an untyped integer constant
	_ = arr[x]
	_ = arr[y]
}
```

- **Index type:**
    - The index expression can be any integer type (`int`, `int32`, `int64`, `uint`, etc.) or an untyped integer constant.
    - Go does **not require a specific integer type** for an index.
- **Why `int` (signed)?**
    - Indexes get used in plain arithmetic all the time, so Go keeps the type signed to avoid underflow.
    - With an unsigned type, `len(s) - 1` on an empty slice wraps to a huge number. With a signed `int` it is just `-1`, which is what you would expect.
- **Bounds checking:** `arr[i]` is checked at runtime and **panics** if `i` is out of bounds or negative. The compiler may remove the check when it can prove at compile time that the index is always valid.
- **Length and arithmetic:** `len(x)` and `cap(x)` return `int` (signed).
    - Because it is signed, a reverse loop like `for i := len(s) - 1; i >= 0; i--` just works, even on an empty slice.

### Strings are different

- **Go:** `s[i]` indexes the underlying **bytes** and returns a `byte` (`uint8`), not a rune.
  A Go string is just a read-only sequence of bytes (usually UTF-8), so `s[0]` on `"héllo"`
  gives you the first *byte*, which may be only half of a character.
- **Rust:** you *cannot* write `s[i]` on a `str`/`String` at all. `str` doesn't implement
  `Index<usize>`. The language refuses, on purpose, so you can't accidentally split a UTF-8
  character in half. Instead you either slice a byte range, `&s[0..4]` (which *panics* if the
  range doesn't land on a character boundary), or iterate explicitly with `.chars()` (Unicode
  scalar values) or `.bytes()` (raw bytes).

So yes, indexing has a type. It's cool to see these design decisions.
One sentence in a book, and now I notice it everywhere.
