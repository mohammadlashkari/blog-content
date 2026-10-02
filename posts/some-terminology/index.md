---
title: Some Terminology
slug: some-terminology
description: The words we use interchangeably, and what they actually mean
language: en
is_favorite: false
tags:
  - general
published_at:
---

In our daily work we use some words interchangeably, and people still get what we mean and it's fine. But I think it's worth knowing the difference. Some of them are:

- declare, define, initialize
- infer, convert, cast, coerce
- argument, parameter
- Unicode, code point, UTF-8
- race condition, data race

Let's check each one.

## Variable declaration, definition, initialization

A **declaration** tells the language "this name exists, and here's its type". It doesn't necessarily reserve any storage. In C, `extern int foo;` declares `foo` without allocating anything for it; it only promises that `foo` exists somewhere.

A **definition** is what actually reserves the memory. Where that memory ends up depends on the variable: a local usually lives on the stack, a global lives in static storage. And most of the time, one line does both jobs at once: `int foo;` is a definition and a declaration.

**Initialization** gives a variable its first value, at the point it's defined.

**Assignment** gives an existing variable a new value, any time after it's defined.

A definition either comes with an initializer, or it doesn't, in which case you assign a value later.

```c
// definition with initialization
int foo = 23;

// definition without initialization
int bar;
// assignment
bar = 42;
```

What happens if you read a variable before it ever gets a value depends on the language:

- **C**: a local variable is not initialized, and reading it is undefined behavior. Globals and `static` variables start at zero.
- **Go**: there is no such thing as an uninitialized variable. Every variable starts at its **zero value**: `0`, `""`, `false`, `nil`, and so on.
- **Rust**: the compiler tracks whether a binding is definitely initialized. `let x;` followed later by `x = 5;` is fine, but reading `x` in between is a compile error.

## Infer, convert, cast, coerce

## Argument and parameter

A **parameter** is the name in the function's definition. It belongs to the function scope and is bound to a new value on every call. An **argument** is the value you pass at the call site.

```go
// x and y are parameters
func add(x, y int) int {
	return x + y
}

add(2, 3)  // 2 and 3 are arguments
add(x, y)  // x and y are arguments
```

Old textbooks say **formal parameter** for the first one and **actual parameter** for the second, which is probably where the mixup started.

## Unicode, code point, UTF-8

I won't explain how Unicode works in detail, just the difference between these three words. (There are pretty good YouTube videos you can find)

**Unicode** is a big table. It contains basically every character you can think of: ASCII letters, Farsi, Chinese, emoji and gives each one a unique number.
That number is the **code point**. It's usually written in hex with a `U+` prefix:
- `A` is `U+0041`
- `س` is `U+0633`
- `😊` is `U+1F60A`

Unicode doesn't say how that number is stored in memory or in a file, so Unicode itself is not an encoding.
That's what **UTF-8** does. It's an encoding: a rule for turning a code point into bytes. It's variable width, so a code point takes 1 to 4 bytes depending on how big it is:
```
char   code point   UTF-8 bytes
A      U+0041       41
س      U+0633       D8 B3
😊     U+1F60A      F0 9F 98 8A
```

- ASCII characters take 1 byte, the same byte as in ASCII, so every ASCII file is already valid UTF-8.
- Some characters are made by combining code points, for example emojis with skin tones: `👍🏽` is `👍` (`U+1F44D`) + skin tone (`U+1F3FD`), so it takes 8 bytes not 4.

So: Unicode is the table, a code point is a number in that table, and UTF-8 is the way of writing that number as bytes.

```txt
      Unicode table          memory (UTF-8)
  ┌──────┬────────────┐
  │ char │ code point │
  ├──────┼────────────┤
  │  A   │ U+0041     │ ───> [41]
  │  س   │ U+0633     │ ───> [D8][B3]
  │  😊  │ U+1F60A    │ ───> [F0][9F][98][8A]
  │  👍  │ U+1F44D    │ ───> [F0][9F][91][8D] ┐
  │  🏽  │ │ U+1F3FD    │ ───> [F0][9F][8F][BD] ┘ 👍🏽 = 8 bytes
  └──────┴────────────┘

```

## Race condition and data race

