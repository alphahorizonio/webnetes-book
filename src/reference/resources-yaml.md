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

A processor defines a virtual node. It references the runtimes and capabilities of the virtual node by their label in it's spec.

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

A signaler signals connections as described in [unisockets](https://github.com/alphahorizonio/unisockets#components).

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

A STUN Server creates peer-to-peer connections by hole-punching NATs.

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

A TURN server relays messages when peer-to-peer connections are not possible.

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

A network references the signalers, STUN & TURN servers by their label and thus defines a overlay network by which the [networking system](../architecture/networking.md) can connect nodes.

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

A network interface allows connecting to a network by a prefix, which is the first three octets of an IPv4 address.

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

A tracker is a WebTorrent tracker. It tracks the files which are currently being seeded and by whom they are being seeded.

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

A repository references the trackers, STUN servers & TURN servers by their labels and thus creates a file repository which can be used to seed & add files.
