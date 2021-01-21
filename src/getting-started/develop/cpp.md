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
