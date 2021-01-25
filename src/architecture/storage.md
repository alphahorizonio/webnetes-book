# Storage

The Webnetes storage/registry stack is build on [WebTorrent](https://en.wikipedia.org/wiki/WebTorrent), an implementation of the BitTorrent protocol based on WebRTC. It is used to make binaries available for workloads. The stack is configured using resources.

To support flexible/private deployments, Webnetes enables using multiple WebTorrent configurations using repositories. Binaries are referenced using an info hash or a magnet link, just like with BitTorrent.

For more information on how to use the storage system, consult the [getting started guide](../getting-started) or the [resources YAML reference](../reference/resources-yaml.md).

For more information on how the storage system works, consult the [file repository documentation](https://alphahorizonio.github.io/webnetes/classes/filerepository.html).
