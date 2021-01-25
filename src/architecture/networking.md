# Networking

The Webnetes network stack is build on [WebRTC](https://en.wikipedia.org/wiki/WebRTC), a peer-to-peer real-time communication technology. This stack is used universally for both [management](./management.md) and inter-workload communication and exposed using [unisockets](https://github.com/alphahorizonio/unisockets), a universal Berkeley sockets implementation for WebRTC. The stack is configured using resources.

For more information on how to use the networking system, consult the [getting started guide](../getting-started) or the [resources YAML reference](../reference/resources-yaml.md).

For more information on how the networking system works, consult the [unisockets documentation](https://alphahorizonio.github.io/unisockets/) and in particular the [unisockets transporter documentation](https://alphahorizonio.github.io/unisockets/classes/transporter.html).
