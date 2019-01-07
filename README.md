Rust-PHF
=========

[![CircleCI](https://circleci.com/gh/sfackler/rust-phf.svg?style=shield)](https://circleci.com/gh/sfackler/rust-phf) [![Latest Version](https://img.shields.io/crates/v/phf.svg)](https://crates.io/crates/phf)

[Documentation](https://docs.rs/phf/0.7.21/phf)

Rust-PHF is a library to generate efficient lookup tables at compile time using
[perfect hash functions](http://en.wikipedia.org/wiki/Perfect_hash_function).

It currently uses the [CHD algorithm](http://cmph.sourceforge.net/papers/esa09.pdf) and can generate
a 100,000 entry map in roughly .4 seconds when compiling with optimizations.

Usage
=====

PHF data structures can be constructed via either the compiler plugins in the `phf_macros` crate or
code generation supported by the `phf_codegen` crate. Compiler plugins are not a stable part of Rust
at the moment, so `phf_macros` can only be used with nightlies.

phf_macros
===========

```rust
#![feature(plugin)]
#![plugin(phf_macros)]

extern crate phf;

#[derive(Clone)]
pub enum Keyword {
    Loop,
    Continue,
    Break,
    Fn,
    Extern,
}

static KEYWORDS: phf::Map<&'static str, Keyword> = phf_map! {
    "loop" => Keyword::Loop,
    "continue" => Keyword::Continue,
    "break" => Keyword::Break,
    "fn" => Keyword::Fn,
    "extern" => Keyword::Extern,
};

pub fn parse_keyword(keyword: &str) -> Option<Keyword> {
    KEYWORDS.get(keyword).cloned()
}
```

phf_codegen
===========

build.rs

```rust
extern crate phf_codegen;

use std::env;
use std::fs::File;
use std::io::{BufWriter, Write};
use std::path::Path;

fn main() {
    let path = Path::new(&env::var("OUT_DIR").unwrap()).join("codegen.rs");
    let mut file = BufWriter::new(File::create(&path).unwrap());

    write!(&mut file, "static KEYWORDS: phf::Map<&'static str, Keyword> = ").unwrap();
    phf_codegen::Map::new()
        .entry("loop", "Keyword::Loop")
        .entry("continue", "Keyword::Continue")
        .entry("break", "Keyword::Break")
        .entry("fn", "Keyword::Fn")
        .entry("extern", "Keyword::Extern")
        .build(&mut file)
        .unwrap();
    write!(&mut file, ";\n").unwrap();
}
```

lib.rs

```rust
extern crate phf;

#[derive(Clone)]
enum Keyword {
    Loop,
    Continue,
    Break,
    Fn,
    Extern,
}

include!(concat!(env!("OUT_DIR"), "/codegen.rs"));

pub fn parse_keyword(keyword: &str) -> Option<Keyword> {
    KEYWORDS.get(keyword).cloned()
}
```
