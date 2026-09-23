# grafana

Runs Grafana with Loki, Mimir and Tempo already provisioned as data sources
and linked to each other: from a log line to its trace, from a trace to its
logs, and a service map from span metrics.

```yaml
- name: Grafana
  hosts: observability
  become: true
  roles:
    - role: eugene_panin.observability.grafana
      vars:
        grafana_bind_address: 10.0.0.10
        grafana_admin_password: "{{ vault_grafana_admin_password }}"
        grafana_loki_url: http://10.0.0.10:3100
        grafana_mimir_url: http://10.0.0.10:9009/prometheus
        grafana_tempo_url: http://10.0.0.10:3200
```

## Data sources

Each of `grafana_loki_url`, `grafana_mimir_url` and `grafana_tempo_url` adds
its data source when set, with a fixed uid (`loki`, `mimir`, `tempo`) so
dashboards can refer to them. The links between them are added only when both
ends exist. `grafana_datasources` takes any other data source in Grafana's
provisioning format.

The provisioning file is written with `prune: true`, so a data source removed
from the variables is removed from Grafana.

## Admin password

Grafana reads `admin_password` only on its very first start. After that the
password lives in its database. The role treats `grafana_admin_password` as
the desired state: every run tries it, and resets the password with
`grafana cli` when it does not work.

## Releases side by side

Each release is unpacked into `grafana_install_dir/grafana-<version>` and
`current` points at the wanted one. Changing `grafana_version` unpacks the new
release, moves the link and restarts Grafana; the old release stays on disk.

The archive is checked against the SHA-256 published next to it on
dl.grafana.com before it is unpacked.

## Tested

One scenario runs Loki, Mimir and Tempo through this collection's roles and
Grafana on top, upgrades Grafana and changes the admin password, then asks
Grafana to health-check each data source. The old password must be refused
and the new one accepted.

## Variables

| Variable | Default | Purpose |
|---|---|---|
| `grafana_bind_address` | required | Address the web server binds to |
| `grafana_admin_password` | required | Desired admin password, from a vault |
| `grafana_version` | `13.2.2` | Exact version |
| `grafana_loki_url`, `grafana_mimir_url`, `grafana_tempo_url` | `""` | Backends to provision |
| `grafana_datasources` | `[]` | Further data sources, provisioning format |
| `grafana_ini` | `{}` | Merged into grafana.ini, section by section |
| `grafana_http_port` | `3000` | Port |

## Notes

- No TLS of its own; bind it to a private address or put a proxy in front.
- Anonymous access and sign-up are off, and so are update checks and
  usage reporting.
