# Changelog

## 0.2.0 — breaking

### Removed: `inject-scenarios`

The `inject-scenarios` skill is gone. It documented `rockfish.labs.scenarios`, which
called the remote `manta` service — and that module **no longer exists in the SDK** as of
rockfish 0.79.0. Code written against it fails at import, not at runtime.

Use [`inject-incidents`](skills/inject-incidents/) instead. It does the same job with
`rockfish.agentfuel`, which runs the injection math locally, with no service.

**Migrating.** Swap the dict `config=` payload for the matching typed config:

| `rockfish.labs.scenarios` | `rockfish.agentfuel` |
| --- | --- |
| `{"type": "spike", ...}` | `InstantaneousSpikeIncidentConfig` |
| `{"type": "outage", ...}` | `DataOutageIncidentConfig` |
| `{"type": "shift", ...}` | `SustainedMagnitudeChangeIncidentConfig` |
| `{"type": "ramp", ...}` | `ValueRampIncidentConfig` |

Field names change too: `measurement` → `impacted_measurement`, and the magnitude field
is `absolute_magnitude` (spike, outage), `delta_magnitude` (sustained change), or
`start_magnitude` / `end_magnitude` (ramp — renamed from `start_value` / `end_value`).
Row filters move from ad-hoc keys to
`impacted_metadata_predicate=[MetadataPredicate(col, value)]`.

Ramp takes at least one endpoint: both fields default to `None`, but omitting *both*
raises `ValueError` from the constructor. Supply one and the other falls back to the
dataset's own first or last value in the window.

For the smallest possible diff, `rockfish.agentfuel.scenarios` stays closer to the old
shape: `SpikeConfig` / `OutageConfig` / `ShiftConfig` / `RampConfig` all keep
`timestamp_column`, `measurement`, and `filter`. `SpikeConfig` is a near drop-in — it
also keeps `timestamp` and `magnitude`. The three range types differ: they take
`start_timestamp` / `end_timestamp` plus their own value field (`outage_value`, `delta`,
or `start_value` / `end_value`). These run on a pandas DataFrame via
`inject_scenario(df, config)` — no upload and no connection — rather than on an
uploaded Rockfish dataset.

Agents that routed to `inject-scenarios` by natural language ("inject an anomaly",
"simulate an outage", "scenario injection") need no change — `inject-incidents` carries
those trigger phrases. Only automation that names the skill `inject-scenarios`
explicitly has to be repointed.
