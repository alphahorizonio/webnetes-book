# Bindings

If you want to support something that isn't part of your language's libraries or not supported by your runtime, you can polyfill it using bindings to Webnetes.

There are currently two bindings which do this, in particular to support networking:

- [unisockets](./unisockets.md), a universal Berkeley sockets implementation
- [tinynet](./tinynet.md), a net implementation for Go and TinyGo based on unisockets

These bindings are enabled using [capabilities](../reference/resources-yaml.md#capability).
