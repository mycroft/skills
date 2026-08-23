# Grafana HTTP API cheat sheet — grafana.services.mkz.me (13.2.0)

Paths below are verified live on this instance. Grafana has no single API prefix, so `grafana-api`
passes the path through as given.

## Why REST and not MCP

- Grafana 13.2.0 exposes no MCP endpoint here: `/api/mcp`, `/mcp`, `/api/mcp/sse` all 404 and no
  `mcp*` feature toggle appears in `/api/frontend/settings`. Enabling it upstream is a server-side
  change (`mcpServer` feature toggle) — the user's call, not an agent's. Re-probe after an upgrade.
- Grafana's standalone `mcp-grafana` server is not installed. It would wrap this same API behind an
  extra process holding the same admin token, so it only pays off for a client that cannot run
  shell commands.

## Orientation

| What | Path |
|---|---|
| token identity | `/api/user` |
| what the token may do | `/api/access-control/user/permissions` |
| server version | `/api/health` |
| datasources | `/api/datasources` (uid, type, isDefault) |
| one datasource | `/api/datasources/uid/{uid}` |
| feature toggles etc. | `/api/frontend/settings` |

Datasource uids on this instance: `prometheus` (default), `GjYZXib4k` (Loki), `bdmi7n5gpyw3kc`
(Tempo), `alertmanager`.

## Query data

Preferred — the datasource proxy, `\/api/datasources/proxy/uid/{uid}` + the datasource's own path:

| Datasource | Path | Params |
|---|---|---|
| Prometheus instant | `/api/v1/query` | `query=`, `time=` |
| Prometheus range | `/api/v1/query_range` | `query=`, `start=`, `end=`, `step=` |
| Prometheus metadata | `/api/v1/label/{name}/values`, `/api/v1/series`, `/api/v1/rules` | |
| Loki range | `/loki/api/v1/query_range` | `query=`, `limit=`, `start=`, `end=` (ns epoch) |
| Loki metadata | `/loki/api/v1/labels`, `/loki/api/v1/label/{name}/values` | |
| Alertmanager | `/api/v2/alerts`, `/api/v2/silences` | |

`start`/`end` accept RFC 3339 (`$(date -u -d '-1 hour' +%FT%TZ)`) or epoch seconds for Prometheus;
Loki wants **nanosecond** epochs. Both verified.

Alternative — `POST /api/ds/query` with `{"queries":[{"refId":"A","datasource":{"uid":…,"type":…},
"expr":…,"instant":true}],"from":"now-5m","to":"now"}`. Answers in Grafana data frames
(`.results.A.frames[].schema/.data.values`), which cost several times more tokens to read than the
native Prometheus shape. Worth it only for mixed-datasource or transformed queries.

## Dashboards, folders, annotations

| What | Path |
|---|---|
| search | `/api/search` `query=`, `type=dash-db|dash-folder`, `tag=`, `limit=` |
| one dashboard | `/api/dashboards/uid/{uid}` |
| dashboard versions | `/api/dashboards/uid/{uid}/versions` |
| folders | `/api/folders`, `/api/folders/{uid}` |
| annotations | `/api/annotations` `from=`, `to=`, `limit=`, `dashboardUID=` |
| create/update dashboard | `post /api/dashboards/db` with `{"dashboard":{…},"overwrite":false}` |
| create annotation | `post /api/annotations '{"text":…,"tags":[…],"time":…}'` |

## Alerting

Grafana-managed alerting is empty here — the rules are datasource-managed in Prometheus.

| What | Path |
|---|---|
| Prometheus rules + state | `/api/datasources/proxy/uid/prometheus/api/v1/rules` |
| firing alerts (Alertmanager) | `/api/datasources/proxy/uid/alertmanager/api/v2/alerts` |
| Grafana-managed rules | `/api/v1/provisioning/alert-rules` (0 here) |
| Grafana-managed, Prom view | `/api/prometheus/grafana/api/v1/rules` (0 groups here) |
| Grafana's own Alertmanager | `/api/alertmanager/grafana/api/v2/alerts` (0 here) |
| contact points | `/api/v1/provisioning/contact-points` |

## Recipes

```bash
# what is firing right now (255 rules / 37 groups on this instance)
grafana-api get /api/datasources/proxy/uid/prometheus/api/v1/rules \
  | jq -r '[.data.groups[].rules[] | select(.state=="firing")] | .[] | "\(.name) \(.labels.severity // "")"'

# instant value of one expression
grafana-api get /api/datasources/proxy/uid/prometheus/api/v1/query 'query=count(kube_pod_info{namespace="flux-system"})' \
  | jq -r '.data.result[0].value[1]'

# top series by value
grafana-api get /api/datasources/proxy/uid/prometheus/api/v1/query 'query=topk(5, sum by (pod) (container_memory_working_set_bytes))' \
  | jq -r '.data.result[] | "\(.metric.pod) \(.value[1])"'

# recent logs for a namespace (Loki wants nanosecond epochs)
grafana-api get /api/datasources/proxy/uid/GjYZXib4k/loki/api/v1/query_range \
  'query={namespace="flux-system"}' limit=20 "start=$(( $(date +%s) - 3600 ))000000000" \
  | jq -r '.data.result[].values[][1]'

# which label values exist
grafana-api get /api/datasources/proxy/uid/prometheus/api/v1/label/job/values | jq -r '.data[]'
grafana-api get /api/datasources/proxy/uid/GjYZXib4k/loki/api/v1/labels | jq -r '.data[]'

# dashboards by name, then their panel queries (panels nest inside rows)
grafana-api get /api/search query=loki type=dash-db | jq -r '.[] | "\(.uid) \(.title)"'
grafana-api get /api/dashboards/uid/operational \
  | jq -r '[.dashboard.panels[]? | (., .panels[]?)] | .[] | select(.type!="row")
           | "\(.title): \([.targets[]?|(.expr//.query//empty)]|join(" ; ")|gsub("\\s+";" ")|.[0:90])"'
```

## Gotchas

- Dashboard JSON runs to ~92 KB here; always project with jq instead of reading it whole.
- Panels inside `row` panels are only reachable via `.panels[].panels[]` — a flat `.panels[]` walk
  silently misses about half of a large dashboard (15 top-level vs 14 rows on `operational`).
- Dashboard panel `targets[]` use `.expr` for Prometheus and `.query` for Loki/others.
- Panel queries contain dashboard variables (`$cluster`, `$namespace`); substitute real values before
  running them through the proxy.
- `Watchdog` firing is intentional in kube-prometheus — it is not an incident.
- The token is org Admin (`dashboards:delete`, `datasources:delete`, `folders:delete` all granted);
  there is no undo for a deleted dashboard beyond its version history.
