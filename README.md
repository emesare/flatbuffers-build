# flatbuffers-build

[![Crates.io](https://img.shields.io/crates/v/flatbuffers-build.svg)](https://crates.io/crates/flatbuffers-build)
[![docs.rs](https://img.shields.io/docsrs/flatbuffers-build)](https://docs.rs/flatbuffers-build/latest/flatbuffers_build/)
![Github Workflow](https://github.com/rdelfin/flatbuffers-build/actions/workflows/ci.yaml/badge.svg)
[![MIT](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/rdelfin/flatbuffers-build/blob/main/LICENSE)

This crate provides a set of functions to facilitate compiling flatbuffers to Rust from within
Rust. This is particularly helpful for use in `build.rs` scripts. Please note that for
compatiblity this crate will only support a single version of the `flatc` compiler. Please
check what version that is against whatever version is installed on your system.That said, due
to flatbuffers' versioning policy, it could be ok to mix patch and even minor versions.

## Usage

If you're not sure where to start, look at the
[`flatbuffers-build-example`](https://github.com/rdelfin/flatbuffers-build/tree/main/flatbuffers-build-example)
folder in the repo for an example. However, we'll explain the full functionality here.

As an example, imagine a crate with the following folder structure:
```bash
├── build.rs
├── Cargo.toml
├── schemas
│   ├── example.fbs
│   └── weapon.fbs
└── src
    └── main.rs
```
In order to compile and use the code generated from both `example.fbs` and `weapon.fbs`, first
you need to add `flatbuffers-build` to your build dependencies, as well as a matching version
of `flatbuffers`:
```toml
# Cargo.toml
# [...]
[dependencies]
flatbuffers = "=25.2.10"

[build-dependencies]
flatbuffers-build = "=0.1.0"
# [...]
```

You can then have a very simple `build.rs` as follows:
```rust
use flatbuffers_build::BuilderOptions;

BuilderOptions::new_with_files(["schemas/weapon.fbs", "schemas/example.fbs"])
    .compile()
    .expect("flatbuffer compilation failed");
```

Note here that `weapon.fbs` and `example.fbs` are based on the schemas provided by
`flatbuffers` as an example. The namespace is `MyGame.Sample` and it contains multiple tables
and structs, including a `Monster` table.

This will just compile the flatbuffers and drop them in `${OUT_DIR}/flatbuffers`.
You can then use them in `lib.rs` like so:

```rust
#[allow(warnings)]
mod gen_flatbuffers {
    include!(concat!(env!("OUT_DIR"), "/flatbuffers/mod.rs"));
}

use gen_flatbuffers::my_game::sample::Monster;

fn some_fn() {
    // Make use of `Monster`
}
```


## On file ordering

Unfortunately due to a quirk in the `flatc` compiler the order you provide the `fbs` files does
matter. From some experimentation, the guidance is to always list files _after_ their
dependencies. Otherwise, the resulting `mod.rs` will be unusable. As an example, we have a
`weapon.fbs` and `example.fbs`. Since the latter has an `include` directive for `weapon.fbs`,
it should go after in the list. If you were to put `example.fbs` _before_ `weapon.fbs`, you'd
end up only being able to import the contents of `weapon.fbs` and with compilation errors if
you tried to use any other components.
