# Resources YAML

Webnetes is based on the concept of resources as the primary means of configuration. This is a listing of all available resources. Many examples of resource YAML usage can be found in the `stack.yaml` files in the [repository](https://github.com/alphahorizonio/webnetes/tree/main/examples).

Resources have a few common fields:

- `apiVersion`: The API version, currently always `schema.webnetes.dev/v1alpha1`
- `kind`: The resource type
- `metadata`: Contains a `label` (the node-unique identifier) and a `name` (the human-readable descriptor)
- `spec`: The resource-specific data

This pattern is strongly influenced by [Kubernetes](https://kubernetes.io/).

## Runtime

```yaml
apiVersion: schema.webnetes.dev/v1alpha1
kind: Runtime
metadata:
  name: Go JSSI
  label: jssi_go
spec: {}
```

A runtime adds support for a language, as described in [compute](../architecture/compute.md). The label can be one of:

- `wasi_generic`: For languages using WASI
- `wasi_tinygo`: Extended WASI for TinyGo
- `jssi_go`: For Go using JSSI
- `jssi_tinygo`: For TinyGo using JSSI
- `jssi_teavm`: For Java using JSSI

## Capability

```yaml
apiVersion: schema.webnetes.dev/v1alpha1
kind: Runtime
metadata:
  name: Go JSSI
  label: jssi_go
spec: {}
```

A capability enables imports, as described in [compute](../architecture/compute.md). The label can be one of:

- `net_socket`: Enables creating a socket
- `net_send`: Enables sending from a socket
- `net_receive`: Enables receiving from a socket
- `net_bind`: Enables binding a socket
- `net_listen`: Enables listening on a socket
- `net_accept`: Enables accepting on a socket
- `net_connect`: Enables connecting to a socket

## Processor

```yaml
apiVersion: schema.webnetes.dev/v1alpha1
kind: Processor
metadata:
  name: Felix's iPhone
  label: felixs_iphone
spec:
  runtimes:
    - jssi_go
  capabilities:
    - net_socket
    - net_send
    - net_receive
    - net_bind
    - net_listen
    - net_accept
```

A processor defines a virtual node. It has the following fields:

- `runtimes`: An array of [runtime](#runtime) labels
- `capabilties`: An array of [capability](#capability) labels

## Signaler

```yaml
apiVersion: schema.webnetes.dev/v1alpha1
kind: Signaler
metadata:
  name: Public unisockets Signaling Server
  label: unisockets_public
spec:
  urls:
    - wss://signaler.webnetes.dev
  retryAfter: 1000
```

A signaler signals connections as described in [unisockets](https://github.com/alphahorizonio/unisockets#components). It has the following fields:

- `urls`: An array of [signaling server](https://github.com/alphahorizonio/unisockets#signaling-protocol) URLs. Currently, only the first entry is respected.
- `retryAfter`: An interval in milliseconds after which to try to re-connect to the signaling servers

## STUN Server

```yaml
apiVersion: schema.webnetes.dev/v1alpha1
kind: StunServer
metadata:
  name: Google STUN Server
  label: google
spec:
  urls:
    - stun:stun.l.google.com:19302
```

A STUN Server creates peer-to-peer connections by hole-punching NATs. It has the following fields:

- `urls`: An array of STUN server URLs.

## TURN Server

```yaml
apiVersion: schema.webnetes.dev/v1alpha1
kind: TurnServer
metadata:
  name: Twillio TURN Server (UDP)
  label: twillio_udp
spec:
  urls:
    - turn:global.turn.twilio.com:3478?transport=tcp
  username: myusername
  credential: mypassword
```

A TURN server relays messages when peer-to-peer connections are not possible. It has the following fields:

- `urls`: An array of TURN server URLs
- `username`: The username to authenticate with when connecting to the TURN server
- `credential`: The credential to authenticate with when connecting to the TURN server

## Network

```yaml
apiVersion: schema.webnetes.dev/v1alpha1
kind: Network
metadata:
  name: Public unisockets network
  label: unisockets_public
spec:
  signaler: unisockets_public
  stunServers:
    - google
    - twillio
  turnServers:
    - twillio_udp
    - twillio_tcp
    - twillio_tcp_fallback
```

A network defines a global peer-to-peer overlay network. It has the following fields:

- `signaler`: The label of a [signaler](#signaler) to use
- `stunServers`s: An array of [STUN server](#stun-server) labels
- `turnServers`s: An array of [TURN server](#turn-server) labels

## Network Interface

```yaml
apiVersion: schema.webnetes.dev/v1alpha1
kind: NetworkInterface
metadata:
  name: Go Echo Network
  label: go_echo_network
spec:
  network: unisockets_public
  prefix: 127.19.0
```

A network interface allow connecting to a [network](#network)'s subnet. It has the following fields:

- `network`: The label of the [network](#network) in which the subnet resides. If it does not exist, it will be created.
- `prefix`: The first three octets of the subnet to create/join.

## Tracker

```yaml
apiVersion: schema.webnetes.dev/v1alpha1
kind: Tracker
metadata:
  name: OpenWebTorrent
  label: openwebtorrent
spec:
  urls:
    - wss://tracker.openwebtorrent.com
```

A tracker is a WebTorrent tracker. It tracks the [files](#file) which are currently being seeded and by whom they are being seeded.

## Repository

```yaml
apiVersion: schema.webnetes.dev/v1alpha1
kind: Repository
metadata:
  name: Public WebTorrent
  label: webtorrent_public
spec:
  trackers:
    - openwebtorrent
    - fastcast
  stunServers:
    - google
    - twillio
  turnServers:
    - twillio_udp
    - twillio_tcp
    - twillio_tcp_fallback
```

A repository references the [trackers](#tracker), [STUN servers](#stun-server) & [TURN servers](#turn-server) by their labels and thus creates a file repository which can be used to seed & add files.

## File

```yaml
apiVersion: schema.webnetes.dev/v1alpha1
kind: File
metadata:
  name: Go Echo Server Binary
  label: go_echo_server
spec:
  repository: webtorrent_public
  uri: 83db73bb4b044a05df306330e421b2e3d38849e4
```

A file references a file seeded using WebTorrent in a [repository](#repository) using the `uri` field, which can be either a BitTorrent info hash or a magnet link.

## Arguments

```yaml
apiVersion: schema.webnetes.dev/v1alpha1
kind: Arguments
metadata:
  name: Go Echo Server Configuration
  label: go_echo_server
spec:
  argv:
    - -laddr
    - 127.0.8.1:4206
```

A arguments resource supplies the command line arguments for a [workload](#workload) and thus allows configuring your app without requiring interactivity and without re-compiling the binary.

## Workload

```yaml
apiVersion: schema.webnetes.dev/v1alpha1
kind: Workload
metadata:
  name: Go Echo Server
  label: go_echo_server
spec:
  file: go_echo_server
  runtime: jssi_go
  capabilities:
    - net_socket
    - net_send
    - net_receive
    - net_bind
    - net_listen
    - net_accept
  networkInterface: go_echo_network
  arguments: go_echo_server
  terminalLabel: go_echo_server
  terminalHostNodeId: localhost
```

A workload configures an app's resources. It has the following fields:

- `file`: Specifies the [file resource](#file) to use for the workload (the WASM binary)
- `runtime`: The [runtime](#runtime) for the file
- `capabilities`: The [capabilities](#capability) to add to the workload. These capabilities must also exist on the processor you are deploying to (the node).
- `networkInterface`: The [network interface](#network-interface) to attach
- `terminalLabel`: Unique label by which to identify the [workload](#workload)'s terminal
- `terminalHostNodeId`: The node ID to attach the resource to
