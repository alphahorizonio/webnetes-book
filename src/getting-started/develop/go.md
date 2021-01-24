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

## TCP Chat Server

Webnetes has support for networking. Let's create a TCP chat server in Go with native and WebAssembly targets. The full source code of this example is available on [GitHub](https://github.com/alphahorizonio/webnetes/tree/main/examples/go_chat_server).

### Module

See [Hello, world! Module](#module).

### Source Code

Create `main.go` with the following content:

<details>
	<summary>Go Source</summary>

```go
package main

import (
	"flag"
	"fmt"
	"log"

	"github.com/alphahorizonio/tinynet/pkg/tinynet"
	"github.com/google/uuid"
	"github.com/ugjka/messenger"
)

func main() {
	// Parse flags
	laddr := flag.String("laddr", "127.0.0.1:4206", "Address to listen on")
	flag.Parse()

	// Listen
	lis, err := tinynet.Listen("tcp", *laddr)
	if err != nil {
		log.Fatal("could not listen", err)
	}

	log.Println("Listening on", *laddr)

	// Receive & broadcast
	msgr := messenger.New(0, true)
	for {
		// Accept
		conn, err := lis.Accept()
		if err != nil {
			log.Fatal("could not accept", err)
		}

		// Notify peers of new node
		log.Println("Client connected")
		clientID := uuid.New()
		msgr.Broadcast(fmt.Sprintf("<system>:\tClient %v joined\n", clientID.String()))

		// Receive from clients & send to bus
		go func(innerConn tinynet.Conn) {
			for {
				// Receive
				buf := make([]byte, 1024)
				if n, err := innerConn.Read(buf); err != nil {
					if n == 0 {
						break
					}

					log.Println("could not read from connection, removing connection", err)

					break
				}

				// Send to bus
				msgr.Broadcast(fmt.Sprintf("<%v>:\t%v", clientID.String(), string(buf)))
			}

			// Close client socket
			log.Println("Client disconnected")

			_ = innerConn.Close() // We keep closing to the OS
		}(conn)

		// Receive from bus & broadcast to clients
		go func(innerConn tinynet.Conn) {
			// Subscribe to new messages
			bus, err := msgr.Sub()
			if err != nil {
				log.Println("could not subscribe to broadcasted messages", err)
			}
			defer msgr.Unsub(bus)

			for msg := range bus {
				// Send to client
				if n, err := innerConn.Write([]byte(fmt.Sprintf("%v", msg))); err != nil {
					if n == 0 {
						break
					}

					fmt.Println("could not write to connection, removing connection", err)

					break
				}
			}

			// Close client socket
			log.Println("Client disconnected")

			_ = innerConn.Close() // We keep closing to the OS
		}(conn)
	}
}
```

</details>

It is a very simple TCP server that listens to messages, assigns an ID to clients, and broadcasts the messages to every other client.

### Make Configuration

Create a `Makefile` with the following content:

```Makefile
all: native wasm

native:
	@docker run -v ${PWD}:/src:z -v ${PWD}/.go:/go:z golang sh -c 'cd /src && go build -o out/go/chat_server main.go'

wasm:
	@docker run -v ${PWD}:/src:z -v ${PWD}/.go:/go:z -e GOOS=js -e GOARCH=wasm golang sh -c 'cd /src && go build -o out/go/chat_server.wasm main.go'

clean:
	@rm -rf out

seed: wasm
	@docker run -it -v ${PWD}/out:/out:z --entrypoint=/bin/sh schaurian/webtorrent-hybrid -c "/usr/local/bin/webtorrent-hybrid seed /out/go/*.wasm"
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
$ ./out/go/chat_server
2021/01/24 17:48:43 Listening on 127.0.0.1:4206
```

In a second terminal, connect to the server:

```shell
$ nc localhost 4206
```

The server should now display the following:

```shell
2021/01/24 17:49:07 Client connected
```

If you type something into the second terminal and press <kbd>Enter</kbd>, the server should echo it as follows:

```shell
Hello, reader!
<25b81b38-5ad5-4478-931f-564f2fd313f7>: Hello, reader!
```

Let's open up another terminal:

```shell
$ nc localhost 4206
```

The second terminal should now contain a message like this:

```shell
<system>:       Client 80f9391f-7eea-45f6-9b50-1f0482bee9e7 joined
```

If you type something into the third terminal and press <kbd>Enter</kbd>, the server should echo it to as follows:

```shell
Hello from client 2 in terminal 3!
<80f9391f-7eea-45f6-9b50-1f0482bee9e7>: Hello from client 2 in terminal 3!
```

The second terminal should now also contain your message:

```
<80f9391f-7eea-45f6-9b50-1f0482bee9e7>: Hello from client 2 in terminal 3!
```

The native target works! Now continue to [Distribute](../distribute.md) to learn how to run the WebAssembly target.

## TCP Chat Client

Now that we've got a TCP chat server, let's create a TCP chat client in Go with native and WebAssembly targets. The full source code of this example is available on [GitHub](https://github.com/alphahorizonio/webnetes/tree/main/examples/go_chat_client).

### Module

See [Hello, world! Module](#module).

### Source Code

Create `main.go` with the following content:

<details>
	<summary>Go Source</summary>

```go
package main

import (
	"bufio"
	"flag"
	"fmt"
	"os"
	"sync"

	"github.com/alphahorizonio/tinynet/pkg/tinynet"
)

func main() {
	// Parse flags
	raddr := flag.String("raddr", "127.0.0.1:4206", "Address to connect to")
	flag.Parse()

	// Dial
	conn, err := tinynet.Dial("tcp", *raddr)
	if err != nil {
		fmt.Println("could not dial", err)

		os.Exit(1)
	}

	fmt.Println("Connected to", *raddr)

	var wg sync.WaitGroup
	wg.Add(2)

	// Read from stdin & send
	go func(innerWg *sync.WaitGroup) {
		reader := bufio.NewReader(os.Stdin)
		for {
			out, err := reader.ReadString('\n')
			if err != nil {
				fmt.Println("could not read from stdin", err)

				os.Exit(1)
			}

			if n, err := conn.Write([]byte(out)); err != nil {
				if n == 0 {
					break
				}

				fmt.Println("could not write from connection, removing connection", err)

				break
			}
		}

		fmt.Println("Disconnected")

		if err := conn.Close(); err != nil {
			fmt.Println("could not close connection", err)
		}

		innerWg.Done()
	}(&wg)

	// Receive & print
	go func(innerWg *sync.WaitGroup) {
		for {
			buf := make([]byte, 1038)
			if n, err := conn.Read(buf); err != nil {
				if n == 0 {
					break
				}

				fmt.Println("could not read from connection, removing connection", err)

				break
			}

			fmt.Printf("%v", string(buf))
		}

		fmt.Println("Disconnected")

		if err := conn.Close(); err != nil {
			fmt.Println("could not close connection", err)
		}

		innerWg.Done()
	}(&wg)

	wg.Wait()
}
```

</details>

It is a very simple TCP client that reads a message from standard input and sends it to the server. In another Goroutine, it waits for any messages from the server, and prints them to standard out.

### Make Configuration

Create a `Makefile` with the following content:

```Makefile
all: native wasm

native:
	@docker run -v ${PWD}:/src:z -v ${PWD}/.go:/go:z golang sh -c 'cd /src && go build -o out/go/chat_client main.go'

wasm:
	@docker run -v ${PWD}:/src:z -v ${PWD}/.go:/go:z -e GOOS=js -e GOARCH=wasm golang sh -c 'cd /src && go build -o out/go/chat_client.wasm main.go'

clean:
	@rm -rf out

seed: wasm
	@docker run -it -v ${PWD}/out:/out:z --entrypoint=/bin/sh schaurian/webtorrent-hybrid -c "/usr/local/bin/webtorrent-hybrid seed /out/go/*.wasm"
```

It provides five targets:

- `make` builds both the native & the WebAssembly target
- `make native` builds only the native target
- `make wasm` builds only the WebAssembly target
- `make clean` removes the built targets
- `make seed` seeds the WebAssembly target, which will come in handy later.

All of them use Docker, so there is no need to install any dependencies on the host.

### Next Steps

First, [build and run the TCP chat server](#next-steps-1).

In a second terminal, build the native target:

```shell
$ make native
```

Run the native target:

```shell
$ ./out/go/chat_client
Connected to 127.0.0.1:4206
```

The server should now display the following:

```shell
2021/01/24 18:00:47 Client connected
```

If you type something into the terminal and press <kbd>Enter</kbd>, the server should echo it as follows:

```shell
Hello from the native Go chat client!
<c22c532b-bd68-42d6-b2d3-bf69cf0dce8b>: Hello from the native Go chat client!
```

The native target works! Now continue to [Distribute](../distribute.md) to learn how to run the WebAssembly target.
