# eugene_panin.observability

[![CI](https://github.com/eugene-panin/ansible-collection-observability/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/eugene-panin/ansible-collection-observability/actions/workflows/ci.yml)
[![Galaxy](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fgalaxy.ansible.com%2Fapi%2Fv3%2Fplugin%2Fansible%2Fcontent%2Fpublished%2Fcollections%2Findex%2Feugene_panin%2Fobservability%2F&query=%24.highest_version.version&label=galaxy&color=blue&cacheSeconds=3600)](https://galaxy.ansible.com/ui/repo/published/eugene_panin/observability/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)

Alloy, Loki, Mimir, Tempo and Grafana on plain hosts with systemd.

Which role goes on which host is the playbook's business: Alloy on every host
that should be observed, the backends and Grafana on the hosts that store and
show the data. The roles know each other only through variables. Alloy takes
the addresses it ships to and has no dependency on the backend roles, so it
works just as well pointed at Grafana Cloud or someone else's Loki.

## Roles

| Role | Status | Purpose |
|---|---|---|
| [`grafana_release`](roles/grafana_release/README.md) | done | Installs a Grafana Labs binary, verified against its published checksum |
| [`loki`](roles/loki/README.md) | done | Logs, in one process, on local disk or S3 |
| [`mimir`](roles/mimir/README.md) | done | Metrics, in one process, on local disk or S3 |
| [`tempo`](roles/tempo/README.md) | done | Traces, Tempo 3 monolithic, on local disk or S3 |
| [`grafana`](roles/grafana/README.md) | done | Loki, Mimir and Tempo provisioned as linked data sources |
| [`alloy`](roles/alloy/README.md) | done | Agent that ships host metrics, the journal and OTLP traces to the backends |

## Requirements

- ansible-core >= 2.19
- `community.general` >= 10.0.0, pulled in automatically
- Ubuntu 22.04, Ubuntu 24.04 or Debian 12 with systemd

## Install

From [Galaxy](https://galaxy.ansible.com/ui/repo/published/eugene_panin/observability/):

```bash
ansible-galaxy collection install eugene_panin.observability
```

Or pinned in `requirements.yml`:

```yaml
collections:
  - name: eugene_panin.observability
    version: 0.1.1
```

## Development

```bash
make deps
make lint
make sanity
make test ROLE=grafana_release
make test ROLE=grafana_release SCENARIO='-s guard'
make matrix ROLE=grafana_release
```

CI runs every role on the three distributions and on ansible-core 2.19.

## License

MIT — see [LICENSE](LICENSE).
