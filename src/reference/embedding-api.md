# Embedding API

Embedding the [node](../components/webnetes.md) is easy and [API documentation is available](https://alphahorizonio.github.io/webnetes/classes/node.html).

## Getting the Library

Webnetes is available on npm. You can add it to your app like so:

```shell
yarn add @alphahorizonio/webnetes
# Or with npm
npm i -s @alphahorizonio/webnetes
```

If you don't have a bundler available, we also provide a ready-to-use precompiled script which you can include like so:

```html
<script src="https://unpkg.com/@alphahorizonio/webnetes@0.1.2/dist/webLibIIFE/index.iife.js"></script>
```

If you're using the script, the node will be available as `window.WebnetesNode`. If you used the NPM package, you can import it like so:

```typescript
import { Node } from "@alphahorizonio/webnetes";
```

## Using the Library in the Browser

This should enable you to create a minimal node in the browser:

```typescript
const node = new Node(
  async (nodeId, resource) => {
    console.console.log("Created resource", nodeId, resource);
  },
  async (nodeId, resource) => {
    console.console.log("Deleted resource", nodeId, resource);

    if (resource.kind === "workload") window.location.reload();
  },
  async (frame) => {
    console.console.log("Rejected resource", frame);
  },
  async (id) => {
    console.console.log("Management node acknowledged", id);
  },
  async (id) => {
    console.console.log("Management node joined", id);
  },
  async (id) => {
    console.console.log("Management node left", id);
  },
  async (metadata, spec, id) => {
    console.console.log("Resource node acknowledged", metadata, spec, id);
  },
  async (metadata, spec: INetworkInterfaceSpec, id) => {
    console.console.log("Resource node joined", metadata, spec, id);
  },
  async (metadata, spec, id) => {
    console.console.log("Resource node left", metadata, spec, id);
  },
  async (onStdin: (key: string) => Promise<void>, id) => {
    // TODO: Implement terminal creation
  },
  async (id, msg) => {
    // TODO: Implement terminal write
  },
  async (id) => {
    // TODO: Implement terminal deletion
  },
  (id) => {
    const rawInput = prompt(`Please enter standard input for ${id}\n`);

    if (rawInput) return new TextEncoder().encode(rawInput);

    return null;
  }
);
```

Now, implement the `TODO`s and `open` the node using the configuration resources. You can find the full source code [in the repository](https://github.com/alphahorizonio/webnetes/tree/main/app/webnetes_web). Consult the [API documentation](https://alphahorizonio.github.io/webnetes/classes/node.html) for further resources.

## Using the Library in Node.js

Webnetes also has Node.js support. This should enable you to create a minimal node in this context:

```typescript
const node = new Node(
  async (nodeId, resource) => {
    console.log("Created resource", nodeId, resource);
  },
  async (nodeId, resource) => {
    console.log("Deleted resource", nodeId, resource);

    if (resource.kind === "workload") {
      spawn(process.execPath, process.argv.slice(1), {
        cwd: process.cwd(),
        detached: true,
        env: process.env,
        stdio: "inherit",
      }).unref();

      process.exit(0);
    }
  },
  async (frame) => {
    console.log("Rejected resource", frame);
  },
  async (id) => {
    console.log("Management node acknowledged", id);
  },
  async (id) => {
    console.log("Management node joined", id);
  },
  async (id) => {
    console.log("Management node left", id);
  },
  async (metadata, spec, id) => {
    console.log("Resource node acknowledged", metadata, spec, id);
  },
  async (metadata, spec: INetworkInterfaceSpec, id) => {
    console.log("Resource node joined", metadata, spec, id);
  },
  async (metadata, spec, id) => {
    console.log("Resource node left", metadata, spec, id);
  },
  async (onStdin: (key: string) => Promise<void>, id) => {
    console.log("Creating terminal (STDOUT only)", id);
  },
  async (id, msg) => {
    console.log("Writing to terminal (STDOUT only)", id, msg);
  },
  async (id) => {
    console.log("Deleting terminal", id);
  },
  (id) => {
    console.error("STDIN is not supported on node");

    process.exit(1);
  }
);
```

Now, `open` the node using the configuration resources. You can find the full source code [in the repository](https://github.com/alphahorizonio/webnetes/tree/main/app/webnetes_node). Consult the [API documentation](https://alphahorizonio.github.io/webnetes/classes/node.html) for further resources.
