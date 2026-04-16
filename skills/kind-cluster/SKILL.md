---
name: kind-cluster
description: Start a local kind (Kubernetes IN Docker) cluster with configurable nodes, roles, and container engine.
---

# kind-cluster

Start a local [kind](https://kind.sigs.k8s.io/) cluster with a generated config file, then verify it is healthy.

## Phase 1 — Gather requirements

Ask the user (or infer from context) the following before proceeding:

- **Cluster name** — defaults to `kind` if not specified
- **Number of control-plane nodes** — defaults to 1; set to 3 for HA
- **Number of worker nodes** — defaults to 2; set to 0 for a single-node cluster
- **Container engine** — `docker` (default), `podman`, or `nerdctl`
- **Kubernetes version** — optional; e.g. `v1.30.0`. Omit to use kind's default.
- **Extra port mappings** — optional; list of `hostPort:containerPort` pairs to expose on the control-plane node

If the user just says "start a kind cluster" with no further details, use the defaults and proceed.

## Phase 2 — Verify prerequisites

Check that the required tools are present before generating any config:

```sh
kind version
kubectl version --client
```

Check that the chosen container engine is running:

```sh
# docker (default)
docker info

# podman
podman info

# nerdctl
nerdctl info
```

If a tool is missing, tell the user what to install and stop. Do not attempt to install tools silently.

## Phase 3 — Generate the kind config

Write a `kind-config.yaml` to the current directory (or a path the user specifies).

### Minimal single-node config (no workers)

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: <cluster-name>
nodes:
  - role: control-plane
```

### Standard multi-node config (1 control-plane + N workers)

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: <cluster-name>
nodes:
  - role: control-plane
  - role: worker
  - role: worker   # repeat for each additional worker
```

### HA control-plane config (3 control-plane + N workers)

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: <cluster-name>
nodes:
  - role: control-plane
  - role: control-plane
  - role: control-plane
  - role: worker
  - role: worker
```

### With a specific Kubernetes version

Add `image:` to each node entry:

```yaml
nodes:
  - role: control-plane
    image: kindest/node:v1.30.0@sha256:<digest>
  - role: worker
    image: kindest/node:v1.30.0@sha256:<digest>
```

Look up the correct digest for a given version on the [kind release page](https://github.com/kubernetes-sigs/kind/releases).

### With port mappings on the control-plane node

```yaml
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 30080
        hostPort: 8080
        protocol: TCP
```

## Phase 4 — Create the cluster

Use the engine-appropriate command:

| Engine | Command |
|--------|---------|
| `docker` (default) | `kind create cluster --config kind-config.yaml` |
| `podman` | `KIND_EXPERIMENTAL_PROVIDER=podman kind create cluster --config kind-config.yaml` |
| `nerdctl` | `KIND_EXPERIMENTAL_PROVIDER=nerdctl kind create cluster --config kind-config.yaml` |

Before creating, check whether a cluster with the same name already exists:

```sh
kind get clusters
```

If the name appears in the output, stop and ask the user whether to delete it first:

```sh
kind delete cluster --name <cluster-name>
```

Do not delete an existing cluster without explicit user confirmation. Only call `kind create cluster` once the name is confirmed absent (either it was never there, or the user approved deletion and it has been removed).

## Phase 5 — Verify the cluster

After the cluster is created, confirm it is healthy:

```sh
# Check nodes are Ready
kubectl get nodes --context kind-<cluster-name>

# Check core system pods are Running
kubectl get pods -n kube-system --context kind-<cluster-name>
```

Expected output: all nodes in `Ready` state; all kube-system pods in `Running` or `Completed` state.

## Phase 6 — Report

Tell the user:

1. The kubeconfig context name (`kind-<cluster-name>`)
2. The config file written (path)
3. Node summary (control-plane count, worker count)
4. The command to switch context: `kubectl config use-context kind-<cluster-name>`
5. The command to delete the cluster when done: `kind delete cluster --name <cluster-name>`

## Guidelines

- **Idempotency**: always check for an existing cluster of the same name before creating. Never silently overwrite.
- **Config file**: always write the config to a file and pass it via `--config` rather than using CLI flags alone — this makes the setup reproducible.
- **HA control-plane caveat**: kind HA control-plane (3 nodes) requires an external load balancer container that kind manages automatically, but startup is slower (~2–3 min). Warn the user if they request HA.
- **Podman/nerdctl caveats**:
  - Podman requires rootless mode or the user must be in the `podman` group.
  - nerdctl requires containerd with CNI plugins installed.
  - Both are experimental providers in kind; behaviour may differ from docker.
- **Resource limits**: a cluster with many nodes consumes significant CPU and memory. As a rough guide, each kind node uses ~512 MB RAM. Warn the user if the requested topology exceeds 6 nodes.
- **kubeconfig**: kind automatically merges the new cluster's credentials into `~/.kube/config`. If the user wants an isolated kubeconfig, suggest: `kind create cluster --kubeconfig ./kubeconfig.yaml`.
