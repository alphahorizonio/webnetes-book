# C

## Support

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

It prints `Hello, world!` to standard out.

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
- `make seeds` seeds the WebAssembly target, which will come in handy later.

All of them use Docker, so there is no need to install any dependencies on the host.

### Next Steps

Build the native target:

```shell
$ make
```

Run the native target:

```shell
$ ./build/native/hello_world
Hello, world!
```

The native target works! Now continue to [Distribute](../distribute.md) to learn how to run the WebAssembly target.

[^1]: [WebAssembly threads](https://github.com/WebAssembly/threads) are not yet implemented.
