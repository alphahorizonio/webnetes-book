# Management

Webnetes is based on resources, which are documented in the [Resources YAML Reference](../reference/resources-yaml.md). The management system manages these resources across the cluster using a peer-to-peer client protocol based on the [networking system](./networking.md). This protocol is also used to transmit status messages and node information, such as benchmark scores and network performance.

For more information on how the management system works, two pieces are important: the [resources pipe](https://alphahorizonio.github.io/webnetes/classes/resources.html), which handles resources on the local node and the [peer pipe](https://alphahorizonio.github.io/webnetes/classes/peers.html), which manages remote peer pipes and takes care of synchronization.
