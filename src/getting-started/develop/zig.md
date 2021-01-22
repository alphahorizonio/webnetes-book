# Zig

## Table of Contents

<!-- toc -->

## Language Support Status

| Native | WebAssembly | Networking | Concurrency |
| ------ | ----------- | ---------- | ----------- |
| ✅     | ✅          | ❌[^1]     | ✅          |

## Hello, world!

Let's start by writing a hello world app in Zig with native and WebAssembly targets. The full source code of this example is available on [GitHub](https://github.com/alphahorizonio/webnetes/tree/main/examples/zig_hello_world).

### Source Code

Create `main.zig` with the following content:

```zig
const std = @import("std");

pub fn main() !void {
    const stdout = std.io.getStdOut().outStream();
    try stdout.print("Hello, world!\n", .{});
}
```

It prints `Hello, world!` to standard output.

### Make Configuration

Create a `Makefile` with the following content:

```Makefile
all: native wasm

native:
	@docker run -v ${PWD}:/src:z alphahorizonio/zig-sdk sh -c 'cd /src && zig build-exe --output-dir out main.zig'

wasm:
	@docker run -v ${PWD}:/src:z alphahorizonio/zig-sdk sh -c 'cd /src && zig build-exe --output-dir out -target wasm32-wasi main.zig'
	@docker run -v ${PWD}:/src:z alphahorizonio/wasi-sdk sh -c 'cd /src && wasm-opt --asyncify -O out/main.wasm -o out/main.wasm'

clean:
	@rm -rf out

seed: wasm
	@docker run -it -v ${PWD}/out:/out:z --entrypoint=/bin/sh schaurian/webtorrent-hybrid -c "/usr/local/bin/webtorrent-hybrid seed /out/*.wasm"
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
$ ./out/main
Hello, world!
```

The native target works! Now continue to [Distribute](../distribute.md) to learn how to run the WebAssembly target.

[^1]: No Zig bindings to [unisockets](https://github.com/alphahorizonio/unisockets) have been implemented yet.
