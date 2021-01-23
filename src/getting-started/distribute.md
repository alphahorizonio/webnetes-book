# Distribute

Now that you've got your app's build configuration up and running, you can start deploying your app to Webnetes. Doing so is a pretty simple process:

1. Define your app's resources using a `stack.yaml` file
2. Create a cluster using [webnetesctl](https://webnetes.dev/) or [webnetesctl Lite](https://lite.webnetes.dev/)
3. Invite nodes to your cluster
4. Deploy your app's resources to a desired target node
5. Open your app's terminal and interact with it!

Let's get started!

## Defining Resources

The first step is to define your app's resources. Here you'll configure the app's capabilities, it's network configuration, runtime and binary. Its quite similar to how [Kubernetes](https://kubernetes.io/) handles it's resource definition; if you've used the latter before, you'll quickly get the hang of how to use Webnetes YAML.

You can find a pre-configured `stack.yaml` file for the examples in the corresponding source code; see [alphahorizonio/webnetes/examples](https://github.com/alphahorizonio/webnetes/tree/main/examples) for a list of all examples the. To get a head start, you can use the `stack.yaml` of the example you've used. A typical `stack.yaml` (here of the C TCP Echo Server example) looks something like the following:

<details>
	<summary>YAML Source</summary>

```yaml
apiVersion: schema.webnetes.dev/v1alpha1
kind: Runtime
metadata:
  name: Generic WASI
  label: wasi_generic
spec: {}
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: Capability
metadata:
  name: Creating a socket
  label: net_socket
spec: {}
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: Capability
metadata:
  name: Sending over a socket
  label: net_send
spec: {}
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: Capability
metadata:
  name: Receiving from a socket
  label: net_receive
spec: {}
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: Capability
metadata:
  name: Binding an alias to a socket
  label: net_bind
spec: {}
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: Capability
metadata:
  name: Listen on a socket
  label: net_listen
spec: {}
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: Capability
metadata:
  name: Accept on a socket
  label: net_accept
spec: {}
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: Processor
metadata:
  name: Felix's iPhone
  label: felixs_iphone
spec:
  runtimes:
    - wasi_generic
  capabilities:
    - net_socket
    - net_send
    - net_receive
    - net_bind
    - net_listen
    - net_accept
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: Signaler
metadata:
  name: Public unisockets Signaling Server
  label: unisockets_public
spec:
  urls:
    - wss://signaler.webnetes.dev
  retryAfter: 1000
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: StunServer
metadata:
  name: Google STUN Server
  label: google
spec:
  urls:
    - stun:stun.l.google.com:19302
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: StunServer
metadata:
  name: Twillio STUN Server
  label: twillio
spec:
  urls:
    - stun:global.stun.twilio.com:3478?transport=udp
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: TurnServer
metadata:
  name: Twillio TURN Server (UDP)
  label: twillio_udp
spec:
  urls:
    - turn:global.turn.twilio.com:3478?transport=tcp
  username: f4b4035eaa76f4a55de5f4351567653ee4ff6fa97b50b6b334fcc1be9c27212d
  credential: w1uxM55V9yVoqyVFjt+mxDBV0F87AUCemaYVQGxsPLw=
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: TurnServer
metadata:
  name: Twillio TURN Server (TCP)
  label: twillio_tcp
spec:
  urls:
    - turn:global.turn.twilio.com:3478?transport=tcp
  username: f4b4035eaa76f4a55de5f4351567653ee4ff6fa97b50b6b334fcc1be9c27212d
  credential: w1uxM55V9yVoqyVFjt+mxDBV0F87AUCemaYVQGxsPLw=
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: TurnServer
metadata:
  name: Twillio TURN Server Fallback (TCP)
  label: twillio_tcp_fallback
spec:
  urls:
    - turn:global.turn.twilio.com:443?transport=tcp
  username: f4b4035eaa76f4a55de5f4351567653ee4ff6fa97b50b6b334fcc1be9c27212d
  credential: w1uxM55V9yVoqyVFjt+mxDBV0F87AUCemaYVQGxsPLw=
---
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
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: NetworkInterface
metadata:
  name: C Echo Network
  label: c_echo_network
spec:
  network: unisockets_public
  prefix: 127.19.0
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: Tracker
metadata:
  name: OpenWebTorrent
  label: openwebtorrent
spec:
  urls:
    - wss://tracker.openwebtorrent.com
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: Tracker
metadata:
  name: Fastcast
  label: fastcast
spec:
  urls:
    - wss://tracker.fastcast.nz
---
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
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: File
metadata:
  name: C Echo Server Binary
  label: c_echo_server
spec:
  repository: webtorrent_public
  uri: d1eb90cb38bbffd1705f49e3e68a57d2c104b594
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: Arguments
metadata:
  name: C Echo Server Configuration
  label: c_echo_server
spec:
  argv:
    - -l
    - 127.0.0.1
    - -p
    - 1234
---
apiVersion: schema.webnetes.dev/v1alpha1
kind: Workload
metadata:
  name: C Echo Server
  label: c_echo_server
spec:
  file: c_echo_server
  runtime: wasi_generic
  capabilities:
    - net_socket
    - net_send
    - net_receive
    - net_bind
    - net_listen
    - net_accept
  networkInterface: c_echo_network
  arguments: c_echo_server
  terminalLabel: c_echo_server
  terminalHostNodeId: localhost
```

</details>

You can find a full reference of the available resources in the [Resources YAML Reference](../reference/resources-yaml.md).
