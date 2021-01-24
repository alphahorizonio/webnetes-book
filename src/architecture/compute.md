# Compute

The Webnetes compute stack is build on [WebAssembly](https://en.wikipedia.org/wiki/WebAssembly), a byte-code format. In order to run WebAssembly (or WASM for short), a virtual machine is required. In the case of Webnetes, this is called a runtime; the following are currently supported:

- [WASI](https://wasi.dev/) via [wasmer-js](https://github.com/wasmerio/wasmer-js) — currently used to support C, C++, Rust, Zig, AssemblyScript and TinyGo (`wasm32-wasi` target)
- [Go's wasm_exec](https://github.com/golang/go/blob/master/misc/wasm/wasm_exec.js) — currently used to support Go
- [TinyGo's wasm_exec](https://github.com/tinygo-org/tinygo/blob/release/targets/wasm_exec.js) — currently used to support TinyGo (`wasm` target)
- [TeaVM](http://teavm.org/) — currently used to support Java

In Webnetes, a runtime also supports imports, which are enabled using capabilities. Imports are implemented using either the [WebAssembly System Interface (WASI)](https://wasi.dev/) whenever supported by the runtime or a custom implementation which we call the JavaScript System Interface (JSSI).

Another feature is `stdin`/`stdout` redirection. Using the [networking system](./networking.md), Webnetes separates a workload from it's terminal, and enables attaching remote terminals to workloads anywhere in the cluster.

For more information on how to use the compute system, consult the [getting started guide](../getting-started) or the [Resources YAML Reference](../reference/resources-yaml.md).
