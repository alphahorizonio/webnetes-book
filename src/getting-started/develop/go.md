# Go

## Table of Contents

<!-- toc -->

## Language Support Status

| Native | WebAssembly | Networking | Concurrency |
| ------ | ----------- | ---------- | ----------- |
| ✅     | ✅          | ✅         | ✅          |

Go & [TinyGo](https://tinygo.org/) are supported, but the latter only has limited support for concurrency.

## Hello, world!

Let's start by writing a hello world app in Go with native and WebAssembly targets. The full source code of this example is available on [GitHub](https://github.com/alphahorizonio/webnetes/tree/main/examples/go_hello_world).

### Module

Run the following to initialize your module:

```shell
$ go mod init module github.com/myuser/mymodule
```

Don't forget to change `github.com/myuser/mymodule` to a more appropriate module path.

### Source Code

Create `main.go` with the following content:

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, world!")
}
```

It prints `Hello, world!` to standard output.

### Make Configuration

Create a `Makefile` with the following content:

```Makefile
all: native-go native-tinygo wasm-go wasm-tinygo

native-go:
	@docker run -v ${PWD}:/src:z -v ${PWD}/.go:/go:z golang sh -c 'cd /src && go build -o out/go/hello_world main.go'

native-tinygo:
	@docker run -v ${PWD}:/src:z -v ${PWD}/.go:/root/go:z tinygo/tinygo sh -c 'cd /src && mkdir -p out/tinygo && tinygo build -o out/tinygo/hello_world main.go'

wasm-go:
	@docker run -v ${PWD}:/src:z -v ${PWD}/.go:/go:z -e GOOS=js -e GOARCH=wasm golang sh -c 'cd /src && go build -o out/go/hello_world.wasm main.go'

wasm-tinygo:
	@docker run -v ${PWD}:/src:z -v ${PWD}/.go:/root/go:z tinygo/tinygo sh -c 'cd /src && mkdir -p out/tinygo && tinygo build -heap-size 20M -target wasi -o out/tinygo/hello_world.wasm main.go'
	@docker run -v ${PWD}:/src:z alphahorizonio/wasi-sdk sh -c 'cd /src && wasm-opt --asyncify -O out/tinygo/hello_world.wasm -o out/tinygo/hello_world.wasm'

clean:
	@rm -rf out

seed-go: wasm-go
	@docker run -it -v ${PWD}/out:/out:z --entrypoint=/bin/sh schaurian/webtorrent-hybrid -c "/usr/local/bin/webtorrent-hybrid seed /out/go/*.wasm"

seed-tinygo: wasm-tinygo
	@docker run -it -v ${PWD}/out:/out:z --entrypoint=/bin/sh schaurian/webtorrent-hybrid -c "/usr/local/bin/webtorrent-hybrid seed /out/tinygo/*.wasm"
```

It provides eight targets:

- `make` builds both the native & the WebAssembly target for both Go & TinyGo
- `make native-go` builds only the native target, using Go
- `make native-tinygo` builds only the native target, using TinyGo
- `make wasm-go` builds only the WebAssembly target, using Go
- `make wasm-tinyoo` builds only the WebAssembly target, using TinyGo
- `make clean` removes the built targets
- `make seed-go` seeds the WebAssembly target built with Go, which will come in handy later.
- `make seed-tinygo` seeds the WebAssembly target built with TinyGo, which will come in handy later.

All of them use Docker, so there is no need to install any dependencies on the host.

### Next Steps

Build all targets:

```shell
$ make
```

Run the native Go target:

```shell
$ ./out/go/hello_world
Hello, world!
```

Run the native TinyGo target:

```shell
$ ./out/tinygo/hello_world
Hello, world!
```

The native targets work! Now continue to [Distribute](../distribute.md) to learn how to run the WebAssembly targets.
