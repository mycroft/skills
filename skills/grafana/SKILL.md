---
name: grafana
description: Use when querying metrics or logs, checking what is firing, or reading dashboards on the user's Grafana at grafana.services.mkz.me — "is anything alerting", "what does CPU look like", "which dashboard shows X" — or when a grafana.services.mkz.me URL needs resolving.
---

# Grafana (grafana.services.mkz.me)

Grafana 13.2.0 over a Kubernetes stack. **No MCP server** — `/api/mcp`, `/mcp` and `/api/mcp/sse`
all 404, the `mcpServer` feature toggle is not enabled, and grafana's separate `mcp-grafana` binary
is not installed. The HTTP API is the access path. Token from `pass mcp/grafana` (`glsa_…`, the
`mcp-admin` service account, **Admin** role in org 1), sent as `Authorization: Bearer`.

Use `scripts/grafana-api` from this skill directory; resolve its absolute path once, then reuse it.
Never put the token on a command line or in a file — the script reads `pass mcp/grafana` itself (or
`$GRAFANA_TOKEN` when already exported).

## Query data through the datasource proxy

For metrics and logs, go straight at the datasource: `/api/datasources/proxy/uid/{uid}/…` speaks the
datasource's own API and returns its native JSON.

```bash
grafana-api get /api/datasources/proxy/uid/prometheus/api/v1/query 'query=sum(up)'
grafana-api get /api/datasources/proxy/uid/prometheus/api/v1/query_range \
  'query=rate(node_cpu_seconds_total[5m])' \
  "start=$(date -u -d '-1 hour' +%FT%TZ)" "end=$(date -u +%FT%TZ)" step=300
grafana-api get /api/datasources/proxy/uid/GjYZXib4k/loki/api/v1/query_range \
  'query={namespace="flux-system"}' limit=20
```

Datasource uids here: **`prometheus`** (default), **`GjYZXib4k`** (Loki), `bdmi7n5gpyw3kc` (Tempo),
`alertmanager`. Re-check with `get /api/datasources`.

`POST /api/ds/query` also works but answers in Grafana data frames — schema-plus-columns, far more
tokens to read than Prometheus' `{"data":{"result":[…]}}`. Use it only for mixed or transformed
queries; prefer the proxy otherwise.

## Alerts live in Prometheus, not Grafana

Grafana-managed alerting is empty on this instance (`/api/v1/provisioning/alert-rules` → 0,
`/api/prometheus/grafana/api/v1/rules` → 0 groups). The real rules are datasource-managed:

```bash
# 255 rules across 37 groups; what is actually firing
grafana-api get /api/datasources/proxy/uid/prometheus/api/v1/rules \
  | jq -r '[.data.groups[].rules[] | select(.state=="firing")] | .[] | "\(.name) \(.labels.severity // "")"'
grafana-api get /api/datasources/proxy/uid/alertmanager/api/v2/alerts | jq -r '.[].labels.alertname'
```

`Watchdog` firing is normal — kube-prometheus keeps it lit on purpose.

## Dashboards

50 of them, one folder (`dev`). Search first, then read selectively:
`get /api/search query=flux type=dash-db` → uid, then `get /api/dashboards/uid/{uid}`.

Dashboard JSON is big (up to ~92 KB here) — **never dump it whole.** Panels nest inside rows, so
recurse when extracting:

```bash
grafana-api get /api/dashboards/uid/operational \
  | jq -r '[.dashboard.panels[]? | (., .panels[]?)] | .[] | select(.type!="row")
           | "\(.title): \([.targets[]?|(.expr//.query//empty)]|join(" ; ")|gsub("\\s+";" ")|.[0:90])"'
```

Reading a dashboard's panel queries is usually the fastest way to learn the right PromQL for a
question — copy the expression, then run it through the proxy.

`reference/api.md` has the endpoint table and more recipes.

## Before writing

The token is org Admin: it holds `dashboards:delete`, `datasources:delete` and `folders:delete`, so a
wrong call can destroy dashboards that are provisioned nowhere else. Treat this instance as read-only
unless the user asks for a change, confirm the target uid first, and never delete a dashboard,
folder, or datasource on your own initiative.
