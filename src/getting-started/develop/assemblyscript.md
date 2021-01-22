# AssemblyScript

## Table of Contents

<!-- toc -->

## Language Support Status

| Native | WebAssembly | Networking | Concurrency |
| ------ | ----------- | ---------- | ----------- |
| ❌[^1] | ✅          | ❌[^2]     | ❌[^3]      |

## Hello, world!

Let's start by writing a hello world app in AssemblyScript. The full source code of this example is available on [GitHub](https://github.com/alphahorizonio/webnetes/tree/main/examples/assemblyscript_hello_world).

### Package

Create `package.json` with the following content:

```json
{
  "name": "@alphahorizonio/assemblyscript_hello_world",
  "version": "0.0.1-alpha1",
  "description": "AssemblyScript webnetes Hello World example",
  "main": "main.ts",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "asbuild:untouched": "asc main.ts --target debug",
    "asbuild:optimized": "asc main.ts --target release",
    "asbuild": "yarn asbuild:untouched && yarn asbuild:optimized"
  },
  "author": "Felix Pojtinger <felix@pojtinger.com>",
  "license": "AGPL-3.0",
  "devDependencies": {
    "assemblyscript": "^0.17.13"
  },
  "dependencies": {
    "as-wasi": "^0.4.4"
  }
}
```

Don't forget to change `@alphahorizonio/assemblyscript_hello_world` to a more appropriate package name.

Now, setup TypeScript by creating `tsconfig.json` with the following content:

```json
{
  "extends": "assemblyscript/std/assembly.json",
  "include": ["./**/*.ts"]
}
```

AssemblyScript requires another config file, so let's create `asconfig.json` with the following content:

```json
{
  "targets": {
    "debug": {
      "binaryFile": "build/untouched.wasm",
      "textFile": "build/untouched.wat",
      "sourceMap": true,
      "debug": true
    },
    "release": {
      "binaryFile": "build/optimized.wasm",
      "textFile": "build/optimized.wat",
      "sourceMap": true,
      "optimize": true
    }
  },
  "options": {}
}
```

Finally, run the following to install the dependencies:

```shell
$ yarn
```

### Source Code

Create `main.ts` with the following content:

```typescript
import { Console } from "as-wasi";
import "wasi";

Console.log("Hello, world!");
```

It prints `Hello, world!` to standard output.

### Make Configuration

Create a `Makefile` with the following content:

```Makefile
all:
	@docker run -v ${PWD}:/src:z node sh -c 'cd /src && yarn && yarn asbuild'
	@docker run -v ${PWD}:/src:z alphahorizonio/wasi-sdk sh -c 'cd /src && wasm-opt --asyncify -O build/optimized.wasm -o build/optimized.wasm'

clean:
	@rm -rf build

seed: all
	@docker run -it -v ${PWD}/build:/build:z --entrypoint=/bin/sh schaurian/webtorrent-hybrid -c "/usr/local/bin/webtorrent-hybrid seed /build/optimized.wasm"
```

It provides three targets:

- `make` builds the WebAssembly target
- `make clean` removes the built target
- `make seed` seeds the WebAssembly target, which will come in handy later.

All of them use Docker, so there is no need to install any dependencies on the host.

### Next Steps

Now continue to [Distribute](../distribute.md) to learn how to run the WebAssembly target.

[^1]: AssemblyScript targets only WebAssembly.<br>
[^2]: No AssemblyScript bindings to [unisockets](https://github.com/alphahorizonio/unisockets) have been implemented yet.<br>
[^3]: AssemblyScript does [not yet support async/await](https://github.com/AssemblyScript/assemblyscript/issues/376).<br>
