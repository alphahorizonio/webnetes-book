# Resources YAML

Webnetes is based on the concept of resources as the primary means of configuration. This is a listing of all available resources. Many examples of resource YAML usage can be found in the `stack.yaml` files in the [repository](https://github.com/alphahorizonio/webnetes/tree/main/examples).

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
