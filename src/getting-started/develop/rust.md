# Rust

## Table of Contents

<!-- toc -->

## Language Support Status

| Native | WebAssembly | Networking | Concurrency |
| ------ | ----------- | ---------- | ----------- |
| ✅     | ✅          | ❌[^1]     | ✅          |

## Hello, world!

Let's start by writing a hello world app in Rust with native and WebAssembly targets. The full source code of this example is available on [GitHub](https://github.com/alphahorizonio/webnetes/tree/main/examples/rust_hello_world).

### Crate

Run the following to initialize your crate:

```shell
$ cargo new --bin rust_hello_world
```

### Source Code

Replace `src/main.rs` with the following content:

```rust
fn main() {
    println!("Hello, world!");
}
```

It prints `Hello, world!` to standard output.

### Make Configuration

Create a `Makefile` with the following content:

```Makefile
all: native wasm

native:
	@docker run -v ${PWD}:/src:z -v ${PWD}/.cargo:/usr/local/cargo/registry:z alphahorizonio/rust-sdk sh -c 'cd /src && cargo build --release'

wasm:
	@docker run -v ${PWD}:/src:z -v ${PWD}/.cargo:/usr/local/cargo/registry:z alphahorizonio/rust-sdk sh -c 'cd /src && cargo build --release --target wasm32-wasi'
	@docker run -v ${PWD}:/src:z alphahorizonio/wasi-sdk sh -c 'cd /src && wasm-opt --asyncify -O target/wasm32-wasi/release/rust_hello_world.wasm -o target/wasm32-wasi/release/rust_hello_world.wasm'

clean:
	@rm -rf target

seed: clean wasm
	@docker run -it -v ${PWD}/target:/target:z --entrypoint=/bin/sh schaurian/webtorrent-hybrid -c "/usr/local/bin/webtorrent-hybrid seed /target/wasm32-wasi/release/*.wasm"
```

It provides five targets:

- `make` builds both the native & the WebAssembly target
- `make native` builds only the native target
- `make wasm` builds only the WebAssembly target
- `make clean` removes the built targets
- `make seed` seeds the WebAssembly target, which will come in handy later.

All of them use Docker, so there is no need to install any dependencies on the host.

### Next Steps

Build the native target:

```shell
$ make native
```

Run the native target:

```shell
$ ./target/release/rust_hello_world
Hello, world!
```

The native target works! Now continue to [Distribute](../distribute.md) to learn how to run the WebAssembly target.

[^1]: No Rust bindings to [unisockets](https://github.com/alphahorizonio/unisockets) have been implemented yet.
