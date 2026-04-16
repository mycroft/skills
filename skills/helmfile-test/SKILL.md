---
name: helmfile-test
description: Test apps locally using helmfile — bootstrap a kind cluster, then diff or apply releases.
---

# helmfile-test

Deploy and test Helm-based applications locally using [helmfile](https://helmfile.readthedocs.io/). Supports previewing diffs or applying releases to a local kind cluster.

## Phase 1 — Gather context

Before doing anything, determine:

- **helmfile path** — the `helmfile.yaml` (or `helmfile.d/` directory) to use. Default: `./helmfile.yaml` in the current directory.
- **Environment** — helmfile environment to target (e.g. `dev`, `staging`). Default: helmfile's own default environment.
- **Selector** — optional label selector to target specific releases (e.g. `app=my-service`). If not specified, all releases are targeted.
- **Action** — `diff` (preview changes, no cluster modification) or `apply` (deploy releases to the cluster).
- **Cluster** — whether to use an existing kubeconfig context or bootstrap a fresh local kind cluster first.

If the user just says "test my app" or "deploy to kind" with no further details, ask only what is strictly needed (action and any selector). Use sensible defaults for everything else.

## Phase 2 — Verify prerequisites

Check that required tools are available:

```sh
helmfile version
helm version --short
kubectl version --client
```

If `helmfile` or `helm` is missing, tell the user what to install and stop. Do not install tools silently.

## Phase 3 — Bootstrap kind cluster (optional)

If the user wants a fresh local cluster, invoke the **kind-cluster** skill at this point.

> Use the skill: `kind-cluster`
>
> Pass any cluster preferences the user mentioned (name, node count, Kubernetes version). If none were mentioned, use the kind-cluster skill's defaults (1 control-plane, 2 workers).

Once the cluster is up, confirm the kubectl context is set to `kind-<cluster-name>` before continuing:

```sh
kubectl config current-context
```

If the user already has a cluster and kubeconfig context ready, skip this phase and proceed directly to Phase 4.

## Phase 4 — Run helmfile

### Diff (preview only — no changes applied)

```sh
helmfile \
  --file <helmfile-path> \
  [--environment <env>] \
  [--selector <selector>] \
  diff
```

Read the diff output carefully and summarise:

- Which releases would be **created**, **updated**, or **deleted**
- For updates: the most significant resource-level changes (image tags, replica counts, config values, new/removed resources)
- Any releases that are already in sync (no changes)

Present the summary in a concise table:

```
| Release       | Namespace  | Change        | Notable diff              |
|---------------|------------|---------------|---------------------------|
| my-service    | default    | update        | image tag v1.2 → v1.3     |
| redis         | cache      | no change     | —                         |
| ingress-nginx | infra      | create        | new release               |
```

After the summary, ask the user: **"Apply these changes?"** before proceeding to apply — unless the user already requested apply from the start.

### Apply (deploy changes to the cluster)

```sh
helmfile \
  --file <helmfile-path> \
  [--environment <env>] \
  [--selector <selector>] \
  apply
```

Monitor output for errors. After the command completes:

```sh
# Check all targeted namespaces for pod health
kubectl get pods --all-namespaces --context <context>
```

If any pods are in `CrashLoopBackOff`, `Error`, or `Pending` state, fetch their logs and events to help diagnose:

```sh
kubectl describe pod <pod-name> -n <namespace> --context <context>
kubectl logs <pod-name> -n <namespace> --context <context> --tail=50
```

## Phase 5 — Report

After a diff, report:
- Summary table (see Phase 4)
- Any warnings from helm/helmfile output (deprecated APIs, missing values, skipped releases)
- Suggested next step: apply command the user can run, or offer to apply now

After an apply, report:
- Which releases were installed/upgraded/unchanged
- Pod health summary across targeted namespaces
- Any errors encountered and likely root cause
- If a kind cluster was created: reminder of how to delete it when done:
  ```sh
  kind delete cluster --name <cluster-name>
  ```

## Guidelines

- **Never apply without confirmation** if the user only asked for a diff. Always show the diff summary first and explicitly ask before applying.
- **Selector tip**: when the user names a specific app (e.g. "just test the API"), translate that into a `--selector` flag rather than running against all releases.
- **Helmfile state backend**: if helmfile is configured with a remote state backend and the cluster is fresh, the state bucket/table may not exist yet. Warn the user if helmfile exits with a state initialisation error and suggest `helmfile apply --skip-deps` or checking the state config.
- **Helm diff plugin**: `helmfile diff` requires the [helm-diff](https://github.com/databus23/helm-diff) plugin. If it is missing, tell the user:
  ```sh
  helm plugin install https://github.com/databus23/helm-diff
  ```
- **Values files**: if a referenced values file is missing, helmfile will fail before rendering. Surface this clearly — do not let it look like a Kubernetes error.
- **Cleanup reminder**: always remind the user to delete the kind cluster when they are done testing if one was created during this session.
