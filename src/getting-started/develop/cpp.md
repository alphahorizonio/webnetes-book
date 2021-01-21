# C++

## Table of Contents

<!-- toc -->

## Language Support Status

| Native | WebAssembly | Networking | Concurrency |
| ------ | ----------- | ---------- | ----------- |
| ✅     | ✅          | ✅         | ❌[^1]      |

## Hello, world!

Let's start by writing a hello world app in C++ with native and WebAssembly targets. For C++, the process is pretty much the same as for [C](./c.md). The full source code of this example is available on [GitHub](https://github.com/alphahorizonio/webnetes/tree/main/examples/cpp_hello_world).

### Source Code

Create `main.cpp` with the following content:

```cpp
#include <iostream>

int main() {
  std::cout << "Hello, world!" << std::endl;

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
add_executable(hello_world "main.cpp")

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

See [C Hello, world! Make Configuration](./c.md#make-configuration).

### Next Steps

See [C Hello, world! Next Steps](./c.md#next-steps).

## TCP Echo Server

Webnetes has support for networking. Let's create a TCP echo server in C++ with native and WebAssembly targets. The full source code of this example is available on [GitHub](https://github.com/alphahorizonio/webnetes/tree/main/examples/cpp_echo_server).

### Source Code

Create `main.cpp` with the following content:

<details>
	<summary>C++ Source</summary>

```cpp
extern "C" {
#include "unisockets.h"
}
#include <arpa/inet.h>
#include <cstdint>
#include <iostream>
#include <iterator>
#include <netinet/in.h>
#include <stdexcept>
#include <stdlib.h>
#include <string.h>
#include <string>
#include <sys/socket.h>
#include <unistd.h>

const int BUFLEN_IN = 1024;
const int BUFLEN_OUT = 1038;

int main(int argc, char *argv[]) {
  // Flags
  std::string listen_host = "127.0.0.1";
  int listen_port = 1234;

  int opt;
  while ((opt = getopt(argc, argv, "l:p:")) != -1) {
    switch (opt) {
    case 'l':
      listen_host = optarg;

      optind--;

      break;

    case 'p':
      listen_port = std::stoi(optarg);

      break;

    default:
      std::cout << "Usage: " << argv[0] << " -l HOST -p PORT" << std::endl;

      return EXIT_FAILURE;
    }
  }

  // Address
  sockaddr_in server_address;
  server_address.sin_family = AF_INET;
  server_address.sin_port = htons(listen_port);
  if (inet_pton(AF_INET, listen_host.c_str(), &server_address.sin_addr) == -1) {
    std::cout << "[ERROR] Could not parse IP address: " << strerror(errno)
              << std::endl;

    return EXIT_FAILURE;
  }
  std::string server_address_readable =
      listen_host + ":" + std::to_string(listen_port);

  // Socket
  int server_socket;
  if ((server_socket = socket(AF_INET, SOCK_STREAM, 0)) == -1) {
    std::cout << "[ERROR] Could not create socket: " << strerror(errno)
              << std::endl;

    return EXIT_FAILURE;
  }

  // Bind
  if ((bind(server_socket, reinterpret_cast<sockaddr *>(&server_address),
            sizeof(server_address))) == -1) {
    std::cout << "[ERROR] Could not bind to socket: " << strerror(errno)
              << std::endl;

    return EXIT_FAILURE;
  }

  // Listen
  if ((listen(server_socket, 5)) == -1) {
    std::cout << "[ERROR] Could not listen on socket: " << strerror(errno)
              << std::endl;

    return EXIT_FAILURE;
  }

  std::cout << "[INFO] Listening on " << server_address_readable << std::endl;

  // Accept loop
  for (;;) {
    std::cout << "[DEBUG] Accepting on " << server_address_readable
              << std::endl;

    // Accept
    int client_socket;
    sockaddr_in client_address;
    socklen_t client_address_length;

    if ((client_socket = accept(
             server_socket, reinterpret_cast<sockaddr *>(&client_address),
             &(client_address_length = sizeof(client_address)))) == -1) {
      std::cout << "[ERROR] Could not accept, continuing: " << strerror(errno)
                << std::endl;
    }

    char client_host[INET_ADDRSTRLEN];
    inet_ntop(AF_INET, &client_address.sin_addr, client_host,
              sizeof(client_host));
    std::string client_address_readable =
        std::string(client_host) + ":" +
        std::to_string(client_address.sin_port);

    std::cout << "[INFO] Accepted client " << client_address_readable
              << std::endl;

    // Receive loop
    for (;;) {
      std::cout << "[DEBUG] Waiting for client " << client_address_readable
                << " to send" << std::endl;

      // Receive
      int received_message_length = 1;
      char received_message[BUFLEN_IN] = "";
      if ((received_message_length =
               recv(client_socket, &received_message, BUFLEN_IN, 0)) == -1) {
        std::cout << "[ERROR] Could not receive from client "
                  << client_address_readable
                  << ", dropping message: " << strerror(errno) << std::endl;
      } else if (received_message_length == 0) {
        break;
      }

      std::cout << "[DEBUG] Received " << received_message_length
                << " bytes from " << client_address_readable << std::endl;

      // Process
      char sent_message[BUFLEN_OUT] = "";
      sprintf((char *)(&sent_message), "You've sent: %s", received_message);
      sent_message[BUFLEN_OUT - 1] = '\0';

      // Send
      int sent_message_length = 0;
      if ((sent_message_length =
               send(client_socket, sent_message, BUFLEN_OUT, 0)) == -1) {
        std::cout << "[ERROR] Could not send to client "
                  << client_address_readable
                  << ", dropping message: " << strerror(errno) << std::endl;
      }

      std::cout << "[DEBUG] Sent " << sent_message_length << " bytes to "
                << client_address_readable << std::endl;
    }

    // Shutdown
    if ((shutdown(client_socket, SHUT_RDWR)) == -1) {
      std::cout << "[ERROR] Could not shutdown socket: " << strerror(errno)
                << std::endl;

      return EXIT_FAILURE;
    }
  }

  return EXIT_SUCCESS;
}
```

</details>

It is a very simple TCP server that listens to messages, prefixes them with `You've sent: ` and sends them back to the sender.

Now, copy and paste the [unisockets.h](https://github.com/alphahorizonio/webnetes/blob/main/examples/cpp_echo_server/unisockets.h) header in the same directory as `main.cpp`, which allows networking to function.
