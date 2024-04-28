---
layout: default
title: 001 Word Count
nav_order: 1
parent: Rust
tags: [programming, rust]
---

# Write a program to count words in a txt file

## A simple version using std::str::split

```rust
// filename: simple.rs
use std::collections::HashMap;
use std::error::Error;
use std::io::{self, BufRead, Write};

fn try_main() -> Result<(), Box<dyn Error>> {
    let stdin = io::stdin();
    let stdin = io::BufReader::new(stdin.lock());
    let mut counts: HashMap<String, u64> = HashMap::new();
    for result in stdin.lines() {
        let line = result?;
        for word in line.split(&[' ', '\t', '\n', ':', ',', '?', '.', '(', ')', ';', '!']) {
            let canon = word.to_lowercase();
            *counts.entry(canon).or_insert(0) += 1;
        }
    }

    let mut ordered: Vec<(String, u64)> = counts.into_iter().collect();
    ordered.sort_by(|&(_, cnt1), &(_, cnt2)| cnt1.cmp(&cnt2).reverse());
    for (word, count) in ordered {
        writeln!(io::stdout(), "{} {}", word, count)?;
    }

    Ok(())
}

fn main() {
    if let Err(err) = try_main() {
        eprintln!("{}", err);
        std::process::exit(1);
    }
}
```

## An optimized version

```rust
// filename: optimized.rs
use std::collections::HashMap;
use std::error::Error;
use std::io::{self, Read, Write};

fn increment(counts: &mut HashMap<Vec<u8>, u64>, word: &[u8]) {
    match counts.get_mut(word) {
        Some(count) => *count += 1,
        None => {
            let _ = counts.insert(word.to_vec(), 1);
        }
    }
}

fn try_main() -> Result<(), Box<dyn Error>> {
    let stdin = io::stdin();
    let mut stdin = stdin.lock();
    let mut counts: HashMap<Vec<u8>, u64> = HashMap::new();
    let mut buf = vec![0; 64 * (1 << 10)];
    let mut word_start = None;
    let mut remains = 0;
    loop {
        let nread = stdin.read(&mut buf[remains..])?;
        if nread == 0 {
            if remains > 0 {
                increment(&mut counts, &buf[..remains + 1]);
            }
            break;
        }
        let buf = &mut buf[..nread + remains];

        for i in (0..buf.len()).skip(remains) {
            let b = buf[i];
            if b'A' <= b && b <= b'Z' {
                buf[i] += b'a' - b'A';
            }

            if b == b' '
                || b == b'\n'
                || b == b':'
                || b == b','
                || b == b'?'
                || b == b'.'
                || b == b'('
                || b == b')'
                || b == b';'
                || b == b'!'
            {
                if let Some(start) = word_start.take() {
                    increment(&mut counts, &buf[start..i]);
                }
            } else if word_start.is_none() {
                word_start = Some(i);
            }
        }

        if let Some(ref mut start) = word_start {
            remains = buf.len() - *start;
            buf.copy_within(*start.., 0);
            *start = 0;
        } else {
            remains = 0;
        }
    }

    let mut ordered: Vec<(Vec<u8>, u64)> = counts.into_iter().collect();
    ordered.sort_by(|&(_, cnt1), &(_, cnt2)| cnt1.cmp(&cnt2).reverse());
    for (word, count) in ordered {
        writeln!(io::stdout(), "{} {}", std::str::from_utf8(&word)?, count)?;
    }

    Ok(())
}

fn main() {
    if let Err(err) = try_main() {
        eprintln!("{}", err);
        std::process::exit(1);
    }
}
```

## Benchmark

A sample text file can be downloaded [here](https://github.com/benhoyt/countwords/blob/master/kjvbible.txt).

To evaluate the performances by timing the executions, run the following commands.

```bash
time cargo run -p rust-word-count --bin simple < kjvbible.txt > /dev/null
time cargo run -p rust-word-count --bin optimized < kjvbible.txt > /dev/null
```

Given that the project "rust-word-count" exists under a workspace with two executables and with the following Cargo.toml.

```toml
[package]
name = "rust-word-count"
version = "0.1.0"
edition = "2021"

[dependencies]

[[bin]]
name = "simple"
path = "src/simple.rs"

[[bin]]
name = "optimized"
path = "src/optimized.rs"
```

## References

- [std::str::ptr](https://doc.rust-lang.org/std/primitive.str.html#method.split)
- [Performance comparison: counting words](https://benhoyt.com/writings/count-words/#awk)
