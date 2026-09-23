# alloy

Runs Grafana Alloy on a host and ships three things from it: host metrics to
Mimir, the systemd journal to Loki, and traces that applications on the host
send over OTLP to Tempo.

```yaml
- name: Alloy
  hosts: all
  become: true
  roles:
    - role: eugene_panin.observability.alloy
      vars:
        alloy_mimir_remote_write_url: http://10.0.0.10:9009/api/v1/push
        alloy_loki_push_url: http://10.0.0.10:3100/loki/api/v1/push
        alloy_tempo_otlp_endpoint: 10.0.0.10:4317
```

## What it ships

Each part of the pipeline is rendered only when its destination is set, so
the same role serves a host that ships only logs, or ships everything to
Grafana Cloud instead of this collection's backends.

| Part | Needs | Sends |
|---|---|---|
| Host metrics | `alloy_mimir_remote_write_url` | node exporter metrics, labelled `instance` |
| Journal | `alloy_loki_push_url` | the systemd journal, labelled `job="systemd-journal"` and `instance` |
| OTLP receiver | `alloy_tempo_otlp_endpoint` | traces applications send to `127.0.0.1:4317` or `:4318` |

`alloy_extra_config` is appended to the pipeline verbatim for anything else.

## A broken pipeline never reaches the agent

The rendered file is checked with `alloy fmt` before it replaces the running
one. A pipeline that does not parse fails the run and the agent keeps what it
had.

## Tested

The scenario runs Loki, Mimir and Tempo through this collection's roles and
Alloy in front of them, then checks that data actually arrived: this host's
metrics in Mimir, a line written with `logger` in Loki, and a span sent to
Alloy found in Tempo. It also upgrades Alloy and hands it a pipeline that does
not parse.

## Variables

| Variable | Default | Purpose |
|---|---|---|
| `alloy_mimir_remote_write_url` | `""` | Where host metrics go |
| `alloy_loki_push_url` | `""` | Where the journal goes |
| `alloy_tempo_otlp_endpoint` | `""` | Where traces go, `host:port`, gRPC |
| `alloy_tempo_otlp_insecure` | `true` | Plain gRPC to Tempo |
| `alloy_instance` | `inventory_hostname` | The `instance` label |
| `alloy_otlp_listen_address` | `127.0.0.1` | Where applications send OTLP |
| `alloy_http_listen_address` | `127.0.0.1:12345` | Alloy's own UI and metrics |
| `alloy_version` | `1.19.2` | Exact version |
| `alloy_extra_config` | `""` | Appended to the pipeline |

## Notes

- The agent runs as its own user, in `adm` and `systemd-journal`, which is
  what reading the journal needs.
- Changing `alloy_version` replaces the binary and restarts Alloy.
