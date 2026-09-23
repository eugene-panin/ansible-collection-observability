# eugene_panin.observability

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
| `loki` | not yet | Logs |
| `mimir` | not yet | Metrics |
| `tempo` | not yet | Traces |
| `grafana` | not yet | Dashboards, with Loki, Mimir and Tempo as data sources |
| `alloy` | not yet | Agent that collects from a host and ships to the backends |

## Requirements

- ansible-core >= 2.19
- `community.general` >= 10.0.0, pulled in automatically
- Ubuntu 22.04, Ubuntu 24.04 or Debian 12 with systemd

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
