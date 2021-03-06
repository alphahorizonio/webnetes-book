# C

## Table of Contents

<!-- toc -->

## Language Support Status

| Native | WebAssembly | Networking | Concurrency |
| ------ | ----------- | ---------- | ----------- |
| ✅     | ✅          | ✅         | ❌[^1]      |

## Hello, world!

Let's start by writing a hello world app in C with native and WebAssembly targets. The full source code of this example is available on [GitHub](https://github.com/alphahorizonio/webnetes/tree/main/examples/c_hello_world).

### Source Code

Create `main.c` with the following content:

```c
#include <stdio.h>

int main() {
  printf("Hello, world!\n");

  return 0;
}
```

It prints `Hello, world!` to standard output.

### CMake Configuration

Create `CMakeLists.txt` with the following content:

```CMake
cmake_minimum_required(VERSION 3.13.4)

# WASI Config
if(WASI)
	set(WASI_SDK_PREFIX "/opt/wasi-sdk-12.0")
	set(CMAKE_SYSROOT "${WASI_SDK_PREFIX}/share/wasi-sysroot")
	set(CMAKE_TOOLCHAIN_FILE "${WASI_SDK_PREFIX}/share/cmake/wasi-sdk.cmake")
endif()

# Project
project(hello_world)
add_executable(hello_world "main.c")

# WASI Config
if(WASI)
	set(CMAKE_EXE_LINKER_FLAGS "-Wl,--allow-undefined")
	set(CMAKE_EXECUTABLE_SUFFIX ".wasm")

	add_custom_command(TARGET hello_world
		POST_BUILD
		COMMAND sh -c "wasm-opt --asyncify -O $<TARGET_FILE:hello_world> -o $<TARGET_FILE:hello_world>"
		VERBATIM
	)
endif()
```

It allows compilation of both the native & the WebAssembly target.

### Make Configuration

Create a `Makefile` with the following content:

```Makefile
all: native wasm

native:
	@docker run -v ${PWD}:/src:z silkeh/clang sh -c 'cd /src && mkdir -p build/native && cd build/native && cmake ../.. && make'

wasm:
	@docker run -v ${PWD}:/src:z alphahorizonio/wasi-sdk sh -c 'cd /src && mkdir -p build/wasm && cd build/wasm && cmake -DWASI=true ../.. && make'

clean:
	@rm -rf build

seed: wasm
	@docker run -it -v ${PWD}/build:/build:z --entrypoint=/bin/sh schaurian/webtorrent-hybrid -c "/usr/local/bin/webtorrent-hybrid seed /build/wasm/*.wasm"
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
$ ./build/native/hello_world
Hello, world!
```

The native target works! Now continue to [Distribute](../distribute.md) to learn how to run the WebAssembly target.

## TCP Echo Server

Webnetes has support for networking. Let's create a TCP echo server in C with native and WebAssembly targets. The full source code of this example is available on [GitHub](https://github.com/alphahorizonio/webnetes/tree/main/examples/c_echo_server).

### Source Code

Create `main.c` with the following content:

<details>
	<summary>C Source</summary>

```c
#include "unisockets.h"
#include <arpa/inet.h>
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <strings.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <unistd.h>

#define BACKLOG 5

#define RECEIVED_MESSAGE_BUFFER_LENGTH 1024
#define SENT_MESSAGE_PREFIX "You've sent: "
#define SENT_MESSAGE_BUFFER_LENGTH                                             \
  RECEIVED_MESSAGE_BUFFER_LENGTH + sizeof(SENT_MESSAGE_PREFIX)

int main(int argc, char *argv[]) {
  // Flags parsing
  char *listen_host = "127.0.0.1";
  int listen_port = 1234;

  int opt;
  while ((opt = getopt(argc, argv, "l:p:")) != -1) {
    switch (opt) {
    case 'l':
      listen_host = optarg;

      optind--;

      break;

    case 'p':
      listen_port = atoi(optarg);

      break;

    default:
      fprintf(stderr, "Usage: %s -l HOST -p PORT\n", argv[0]);

      exit(EXIT_FAILURE);
    }
  }

  // Variables
  int server_sock;
  int client_sock;

  struct sockaddr_in server_address;
  struct sockaddr_in client_address;

  socklen_t server_socket_length = sizeof(struct sockaddr_in);

  size_t received_message_length;
  char received_message[RECEIVED_MESSAGE_BUFFER_LENGTH];
  ssize_t sent_message_length;
  char sent_message[SENT_MESSAGE_BUFFER_LENGTH];
  char client_address_readable[sizeof(client_address.sin_addr) +
                               client_address.sin_port + 1];
  char client_address_human_readable[INET_ADDRSTRLEN];

  // Logging
  char server_address_readable[sizeof(listen_host) + sizeof(listen_port) + 1];
  sprintf(server_address_readable, "%s:%d", listen_host, listen_port);

  memset(&server_address, 0, sizeof(server_address));
  memset(&client_address, 0, sizeof(client_address));

  // Create address
  server_address.sin_family = AF_INET;
  server_address.sin_port = htons(listen_port);
  if (inet_pton(AF_INET, listen_host, &server_address.sin_addr) == -1) {
    perror("[ERROR] Could not parse IP address:");

    exit(EXIT_FAILURE);
  }

  // Create socket
  if ((server_sock = socket(AF_INET, SOCK_STREAM, 0)) == -1) {
    perror("[ERROR] Could not create socket:");

    printf("[ERROR] Could not create socket %s\n", server_address_readable);

    exit(EXIT_FAILURE);
  }

  // Bind
  if ((bind(server_sock, (struct sockaddr *)&server_address,
            server_socket_length)) == -1) {
    perror("[ERROR] Could not bind to socket:");

    printf("[ERROR] Could not bind to socket %s\n", server_address_readable);

    exit(EXIT_FAILURE);
  }

  // Listen
  if ((listen(server_sock, BACKLOG)) == -1) {
    perror("[ERROR] Could not listen on socket:");

    printf("[ERROR] Could not listen on socket %s\n", server_address_readable);

    exit(EXIT_FAILURE);
  }

  printf("[INFO] Listening on %s\n", server_address_readable);

  // Accept loop
  while (1) {
    printf("[DEBUG] Accepting on %s\n", server_address_readable);

    // Accept
    if ((client_sock = accept(server_sock, (struct sockaddr *)&client_address,
                              &server_socket_length)) == -1) {
      perror("[ERROR] Could not accept, continuing:");

      continue;
    }

    if (inet_pton(AF_INET, listen_host, &server_address.sin_addr) == -1) {
      perror("[ERROR] Could not parse IP address:");

      continue;
    }

    inet_ntop(AF_INET, &client_address.sin_addr, client_address_human_readable,
              sizeof(client_address_human_readable));

    sprintf(client_address_readable, "%s:%d", client_address_human_readable,
            client_address.sin_port);

    printf("[INFO] Accepted client %s\n", client_address_readable);

    // Receive loop
    received_message_length = 1;
    while (received_message_length) {
      memset(&received_message, 0, RECEIVED_MESSAGE_BUFFER_LENGTH);
      memset(&sent_message, 0, SENT_MESSAGE_BUFFER_LENGTH);

      printf("[DEBUG] Waiting for client %s to send\n",
             client_address_readable);

      // Receive
      received_message_length = recv(client_sock, &received_message,
                                     RECEIVED_MESSAGE_BUFFER_LENGTH, 0);
      if (received_message_length == -1) {
        perror("[ERROR] Could not receive from client:");

        printf("[ERROR] Could not receive from client %s, dropping message\n",
               client_address_readable);

        break;
      }

      if (received_message_length == 0) {
        break;
      }

      printf("[DEBUG] Received %zd bytes from %s\n", received_message_length,
             client_address_readable);

      // Process
      sprintf((char *)&sent_message, "%s%s", SENT_MESSAGE_PREFIX,
              received_message);
      sent_message[SENT_MESSAGE_BUFFER_LENGTH - 1] = '\0';

      // Send
      sent_message_length =
          send(client_sock, sent_message, SENT_MESSAGE_BUFFER_LENGTH, 0);
      if (sent_message_length == -1) {
        perror("[ERROR] Could not send to client:");

        printf("[ERROR] Could not send to client %s, dropping message\n",
               client_address_readable);

        break;
      }

      printf("[DEBUG] Sent %zd bytes to %s\n", sent_message_length,
             client_address_readable);
    }

    printf("[INFO] Client %s disconnected\n", client_address_readable);

    // Shutdown
    if ((shutdown(client_sock, SHUT_RDWR)) == -1) {
      perror("[ERROR] Could not shutdown socket:");

      printf("[ERROR] Could not shutdown socket %s, stopping\n",
             client_address_readable);

      break;
    };
  }

  return 0;
}
```

</details>

It is a very simple TCP server that listens to messages, prefixes them with `You've sent: ` and sends them back to the sender.

Now, copy and paste the [unisockets.h](https://github.com/alphahorizonio/webnetes/blob/main/examples/c_echo_server/unisockets.h) header in the same directory as `main.c`, which allows networking to function.

### CMake Configuration

Create `CMakeLists.txt` with the following content:

```CMake
cmake_minimum_required(VERSION 3.13.4)

# WASI Config
if(WASI)
	set(WASI_SDK_PREFIX "/opt/wasi-sdk-12.0")
	set(CMAKE_SYSROOT "${WASI_SDK_PREFIX}/share/wasi-sysroot")
	set(CMAKE_TOOLCHAIN_FILE "${WASI_SDK_PREFIX}/share/cmake/wasi-sdk.cmake")
endif()

# Project
project(echo_server)
add_executable(echo_server "main.c")

# WASI Config
if(WASI)
	set(CMAKE_C_FLAGS "-DUNISOCKETS_WITH_ALIAS")
	set(CMAKE_EXE_LINKER_FLAGS "-Wl,--allow-undefined")
	set(CMAKE_EXECUTABLE_SUFFIX ".wasm")

	add_custom_command(TARGET echo_server
		POST_BUILD
		COMMAND sh -c "wasm-opt --asyncify -O $<TARGET_FILE:echo_server> -o $<TARGET_FILE:echo_server>"
		VERBATIM
	)
endif()
```

It allows compilation of both the native & the WebAssembly target. Note `-DUNISOCKETS_WITH_ALIAS`, which allows networking to function.

### Make Configuration

See [Hello, world! Make Configuration](#make-configuration).

### Next Steps

Build the native target:

```shell
$ make native
```

Run the native target:

```shell
$ ./build/native/echo_server
[INFO] Listening on 127.0.0.1:1234
[DEBUG] Accepting on 127.0.0.1:1234
```

In a second terminal, connect to the server:

```shell
$ nc localhost 1234
```

The server should now display the following:

```shell
[INFO] Accepted client 127.0.0.1:39644
[DEBUG] Waiting for client 127.0.0.1:39644 to send
```

If you type something into the second terminal and press <kbd>Enter</kbd>, the server should echo it as follows:

```shell
Hello, reader!
You've sent: Hello, reader!
```

The native target works! Now continue to [Distribute](../distribute.md) to learn how to run the WebAssembly target.

## TCP Echo Client

Now that we've got a TCP echo server, let's create a TCP echo client in C with native and WebAssembly targets. The full source code of this example is available on [GitHub](https://github.com/alphahorizonio/webnetes/tree/main/examples/c_echo_client).

### Source Code

Create `main.c` with the following content:

<details>
	<summary>C Source</summary>

```c
#include "unisockets.h"
#include <arpa/inet.h>
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define RECONNECT_TIMEOUT 2

#define SENT_MESSAGE_BUFFER_LENGTH 1024
#define RECEIVED_MESSAGE_PREFIX "You've sent: "
#define RECEIVED_MESSAGE_BUFFER_LENGTH                                         \
  SENT_MESSAGE_BUFFER_LENGTH + sizeof(RECEIVED_MESSAGE_PREFIX)

int main(int argc, char *argv[]) {
  // Flags parsing
  char *connect_host = "127.0.0.1";
  int connect_port = 1234;

  int opt;
  while ((opt = getopt(argc, argv, "c:p:")) != -1) {
    switch (opt) {
    case 'c':
      connect_host = optarg;

      optind--;

      break;

    case 'p':
      connect_port = atoi(optarg);

      break;

    default:
      fprintf(stderr, "Usage: %s -c HOST -p PORT\n", argv[0]);

      exit(EXIT_FAILURE);
    }
  }

  // Variables
  int server_sock;
  struct sockaddr_in server_host;

  ssize_t sent_message_length;
  char read_message[SENT_MESSAGE_BUFFER_LENGTH];
  size_t received_message_length;
  char received_message[RECEIVED_MESSAGE_BUFFER_LENGTH];

  socklen_t server_host_length = sizeof(struct sockaddr_in);

  memset(&server_host, 0, sizeof(server_host));

  // Logging
  char server_address_readable[sizeof(connect_host) + sizeof(connect_port) + 1];
  sprintf(server_address_readable, "%s:%d", connect_host, connect_port);

  // Create address
  server_host.sin_family = AF_INET;
  server_host.sin_port = htons(connect_port);
  if (inet_pton(AF_INET, connect_host, &server_host.sin_addr) == -1) {
    perror("[ERROR] Could not parse IP address:");

    exit(EXIT_FAILURE);
  }

  // Create socket
  if ((server_sock = socket(AF_INET, SOCK_STREAM, 0)) == -1) {
    perror("[ERROR] Could not create socket:");

    printf("[ERROR] Could not create socket %s\n", server_address_readable);

    exit(EXIT_FAILURE);
  }

  // Connect loop
  while (1) {
    printf("[INFO] Connecting to server %s\n", server_address_readable);

    // Connect
    if ((connect(server_sock, (struct sockaddr *)&server_host,
                 server_host_length)) == -1) {
      perror("[ERROR] Could not connect to server:");

      printf("[ERROR] Could not connect to server %s, retrying in %ds\n",
             server_address_readable, RECONNECT_TIMEOUT);

      sleep(RECONNECT_TIMEOUT);

      continue;
    }

    printf("[INFO] Connected to server %s\n", server_address_readable);

    // Read loop
    while (1) {
      memset(&received_message, 0, RECEIVED_MESSAGE_BUFFER_LENGTH);
      memset(&read_message, 0, SENT_MESSAGE_BUFFER_LENGTH);

      printf("[DEBUG] Waiting for user input\n");

      // Read
      fgets(read_message, SENT_MESSAGE_BUFFER_LENGTH, stdin);

      // Send
      sent_message_length =
          send(server_sock, read_message, strlen(read_message), 0);
      if (sent_message_length == -1) {
        perror("[ERROR] Could not send to server:");

        printf("[ERROR] Could not send to server %s, dropping message\n",
               server_address_readable);

        break;
      }

      printf("[DEBUG] Sent %zd bytes to %s\n", sent_message_length,
             server_address_readable);

      printf("[DEBUG] Waiting for server %s to send\n",
             server_address_readable);

      // Receive
      received_message_length = recv(server_sock, &received_message,
                                     RECEIVED_MESSAGE_BUFFER_LENGTH, 0);
      if (received_message_length == -1) {
        perror("[ERROR] Could not receive from server:");

        printf("[ERROR] Could not receive from server %s, dropping message\n",
               server_address_readable);

        break;
      }

      if (received_message_length == 0) {
        break;
      }

      printf("[DEBUG] Received %zd bytes from %s\n", received_message_length,
             server_address_readable);

      // Print
      printf("%s", received_message);
    }

    printf("[INFO] Disconnected from server %s\n", server_address_readable);

    // Shutdown
    if ((shutdown(server_sock, SHUT_RDWR)) == -1) {
      perror("[ERROR] Could not shutdown socket:");

      printf("[ERROR] Could not shutdown socket %s, stopping\n",
             server_address_readable);

      break;
    };
  }

  return 0;
}
```

</details>

It is a very simple TCP client that reads a message from standard input, sends it to the server, waits for a reply, and prints it to standard out.

Now, copy and paste the [unisockets.h](https://github.com/alphahorizonio/webnetes/blob/main/examples/c_echo_client/unisockets.h) header in the same directory as `main.c`, which allows networking to function.

### CMake Configuration

Create `CMakeLists.txt` with the following content:

```CMake
cmake_minimum_required(VERSION 3.13.4)

# WASI Config
if(WASI)
	set(WASI_SDK_PREFIX "/opt/wasi-sdk-12.0")
	set(CMAKE_SYSROOT "${WASI_SDK_PREFIX}/share/wasi-sysroot")
	set(CMAKE_TOOLCHAIN_FILE "${WASI_SDK_PREFIX}/share/cmake/wasi-sdk.cmake")
endif()

# Project
project(echo_client)
add_executable(echo_client "main.c")

# WASI Config
if(WASI)
	set(CMAKE_C_FLAGS "-DUNISOCKETS_WITH_ALIAS")
	set(CMAKE_EXE_LINKER_FLAGS "-Wl,--allow-undefined")
	set(CMAKE_EXECUTABLE_SUFFIX ".wasm")

	add_custom_command(TARGET echo_client
		POST_BUILD
		COMMAND sh -c "wasm-opt --asyncify -O $<TARGET_FILE:echo_client> -o $<TARGET_FILE:echo_client>"
		VERBATIM
	)
endif()
```

It allows compilation of both the native & the WebAssembly target. Note `-DUNISOCKETS_WITH_ALIAS`, which allows networking to function.

### Make Configuration

See [Hello, world! Make Configuration](#make-configuration).

### Next Steps

First, [build and run the TCP echo server](#next-steps-1).

In a second terminal, build the native target:

```shell
$ make native
```

Run the native target:

```shell
$ ./build/native/echo_client
[INFO] Connecting to server 127.0.0.1:1234
[INFO] Connected to server 127.0.0.1:1234
[DEBUG] Waiting for user input
```

The server should now display the following:

```shell
[INFO] Accepted client 127.0.0.1:39644
[DEBUG] Waiting for client 127.0.0.1:39644 to send
```

If you type something into the terminal and press <kbd>Enter</kbd>, the server should echo it as follows:

```shell
Hello, dear reader!
[DEBUG] Sent 20 bytes to 127.0.0.1:1234
[DEBUG] Waiting for server 127.0.0.1:1234 to send
[DEBUG] Received 1038 bytes from 127.0.0.1:1234
You've sent: Hello, dear reader!
[DEBUG] Waiting for user input
```

The native target works! Now continue to [Distribute](../distribute.md) to learn how to run the WebAssembly target.

[^1]: [WebAssembly threads](https://github.com/WebAssembly/threads) are not yet implemented.
