# More

Webnetes can, in addition to the languages documented before, also run any further languages supported by the following runtimes:

- [WASI](https://wasi.dev/) via [wasmer-js](https://github.com/wasmerio/wasmer-js) — currently used to support C, C++, Rust, Zig, AssemblyScript and TinyGo (`wasm32-wasi` target)
- [Go's wasm_exec](https://github.com/golang/go/blob/master/misc/wasm/wasm_exec.js) — currently used to support Go
- [TinyGo's wasm_exec](https://github.com/tinygo-org/tinygo/blob/release/targets/wasm_exec.js) — currently used to support TinyGo (`wasm` target)
- [TeaVM](http://teavm.org/) — currently used to support Java

Even if your language doesn't support any of the ones specified above, adding support for another one is easy (just add your own handler to the [virtual machine](https://github.com/alphahorizonio/webnetes/blob/main/lib/controllers/virtual-machine.ts#L12-L18)). If you wish to have another runtime officially supported, please open a new issue in the [GitHub issue tracker](https://github.com/alphahorizonio/webnetes/issues) and we'll look into it.
