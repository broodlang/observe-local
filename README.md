# observe-local

An [`observe`](../observe) provider that keeps logs, metrics and errors **in the app**,
grouped and bounded, with a page to read them. No database, nothing to configure —
[error_tracker](https://hex.pm/packages/error_tracker)'s idea, for everything the shipper
sends, done the Brood way: one process, one immutable state value, a page rendered from a
snapshot.

The point is to *see what monitoring sees*: every event `observe` would hand a vendor,
exactly as it would hand it — on a laptop with no keys, or in production as a second
opinion next to the vendor.

## Usage

```brood
(observe/start {:provider (observe-local/provider) :app "hive" :env "dev"})
(observe/attach-all)

;; in the router — behind the same guard as the metrics dashboard:
(through [(web/auth/allow-ips-from-env "HIVE_DASHBOARD_IPS")]
  (get "/observe" (observe-local/handler))
  (get "/observe/*path" (observe-local/handler)))
```

Both at once in production:

```brood
(observe/fan-out-provider [(observe-local/provider) (observe-appsignal/provider {…})])
```

## The page

| path | shows |
|---|---|
| `/observe` | counts received (errors, log lines, metrics, batches) and the ten most recent error groups |
| `/observe/errors` | every group — name, first message line, namespace/action, count, last seen — newest first |
| `/observe/errors/<id>` | a group's last 20 occurrences: message, backtrace, tags, params |
| `/observe/logs?level=warn` | the newest 200 lines at that level and above, with their structured meta |
| `/observe/metrics` | every series (name + tags): count, last, p50, p95, max, sum over the recent samples |

Self-contained — inline styles, no assets, no JavaScript, `noindex`.

## What it keeps

| kind | grouped by | bounded by |
|---|---|---|
| errors | name + first line of the message (how error_tracker fingerprints) | 200 groups (least recently seen evicted), 20 occurrences each |
| logs | — | the newest 500 lines |
| metrics | name + tags | 500 series (new ones past that are dropped), 200 samples each |

The caps are `*max-error-groups*`, `*max-occurrences*`, `*max-logs*`, `*max-metric-keys*`,
`*max-samples*`. The folds (`add-error`, `add-log`, `add-metric`, `receive-batch`) and
the queries (`errors`, `error-group`, `logs`, `metrics`, `summary`) are pure over the
state and take a snapshot, so a test never needs the process. A restart forgets everything;
a `store`-backed twin with the same queries is the obvious next step and would replace only
the process, not the page.

## Development

The suite is `tests/observe-local_test.blsp`;
`nest format` before committing.
