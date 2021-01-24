# Resources YAML

Webnetes is based on the concept of resources as the primary means of configuration. This is a listing of all available resources.

## Runtime

A runtime, as described in [compute](../architecture/compute.md). The label can be one of:

- `wasi_generic`, for languages using WASI
- `wasi_tinygo`, extended for TinyGo using WASI
- `jssi_go`, for Go using JSSI
- `jssi_tinygo`, for TinyGo using JSSI
- `jssi_teavm`, for Java using JSSI

```yaml
apiVersion: schema.webnetes.dev/v1alpha1
kind: Runtime
metadata:
  name: Go JSSI
  label: jssi_go
spec: {}
```
