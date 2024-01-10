# 🦀 Rust Panic Analyzer 🔎

## Overview

Rust Panic Analyzer is an audit tool designed to scan your Rust crate or workspace. Its primary function is to identify potential panic points in your codebase, leading you in developing binaries and libraries that are as close to "Panic Free" as possible.

## How does it work?

### Key Identification Patterns

The tool searches for usage of several key patterns in Rust code that are often associated with panic points. These include:

- `panic!`: Direct calls to the `panic!` macro, which causes the program to terminate immediately and provide an error message.
- `unwrap`: Calls to the `.unwrap()` method, often used on `Option` or `Result` types, which will cause a panic if the value is `None` or `Err`.
- `expect`: Similar to `unwrap`, but allows specifying a custom error message.
- `Array Indexing`: Direct indexing into arrays (e.g., arr[index]) without bounds checking, which can panic if the index is out of bounds. (A safer indexing method is `.get()`)
- `unreachable!`: Indicates code that should never be reached; panics if executed.
- `todo!` and `unimplemented!`: Macros indicating incomplete or unimplemented code, which will panic if reached.


## Installation

To start using it, you need to install it first.

```sh
cargo install panic-free-analyzer
```


## Usage Locally:

After installation, you can run the analyzer on your crate or entire workspace. Use the following command:

```sh
cargo panic-analyzer > audit.md
```

Logging audit result to the terminal
```sh
cargo panic-analyzer
```

If you wish to exclude specific crates from your workspace during the analysis, set the `IGNORED_CRATES`` environment variable. Pass the names of the crates you want to exclude, separated by commas:

```sh
IGNORED_CRATES=tests,benches cargo panic-analyzer > audit.md
```

A potential panic is not necessarily bad, sometimes errors are unrecoverable, and we have to panic.
If your panic is intentional, you can add a comment before the line that has the potential panicing code like this:

```rs
pub fn shutdown_server() {
  // @expected: we need this
  panic!("Exited process!")
}
```

The syntax is as the following: `// @expected: description/reason`.

This won't be counted as a potential panic point, but rather an expected panic in a section at the end.


## Pull Requests Audit results via GitHub Actions

You can also hook it up with your CI to post the results on your PRs as a comment which can be very helpful!

```yaml
name: ci

on:
  push:
    branches:
      - main
  pull_request:
jobs:
  panic-free-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions-rust-lang/setup-rust-toolchain@v1

      - name: install panic free analyzer
        run: |
            cargo install panic-free-analyzer

      - name: run panic free analyzer
        run: |
            cargo panic-analyzer > ./audit.md
        env:
          IGNORED_CRATES: e2e_tests,benches

      - name: comment on pull request
        uses: thollander/actions-comment-pull-request@v2
        with:
          filePath: ./audit.md
          comment_tag: rust-code-audit

```

## ⚠️ Limitations:

- As of now, it only currently searches the crates that you develop, and not the dependencies of your crates.
- The analysis is done using regex rather than an AST for simplicity, so you may encounter unidentified potential panic points if the lines were wrapped in certain ways that the regex patterns can't catch.


# Audit Results Example 👇

Below is an example of an audit result generated by the Rust Panic Free Analyzer:


# 🚨 Rust Panic Audit: 82 Potential Panic Points Detected 🚨

## Crate: `vrl`

📊 Total Usages: 37

- 🔎 `expect` usages: 1
- 🎁 `unwrap` usages: 32
- 🚨 `panic` usages: 1
- 🔢 `array_index` usages: 3

## Crate: `jwt_auth`

📊 Total Usages: 31

- 🎁 `unwrap` usages: 29
- 🔢 `array_index` usages: 2

## Crate: `config`

📊 Total Usages: 14

- 🚨 `panic` usages: 3
- 🔎 `expect` usages: 3
- 🎁 `unwrap` usages: 8

## 📌 Expected Annotations

### Crate: `common`

📊 Total Expected Usages: 1

- Reason: "we need this"
- Code: `panic!("Exited process!")`
- Location: `./libs/common/src/lib.rs:18`
