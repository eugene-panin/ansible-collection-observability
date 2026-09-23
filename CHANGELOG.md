# Changelog

All notable changes to this collection are documented here.
This project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added

- `grafana_release` role: installs Loki, Mimir, Tempo or Alloy from its GitHub
  release. Reads both checksum formats Grafana Labs publishes and refuses to
  install an asset it finds no checksum for. Compares the installed version,
  so a version bump upgrades in place, and tells the calling role through
  `grafana_release_changed`.
- `loki` role: Loki with all components in one process, chunks and the TSDB
  index on local disk or in an S3-compatible bucket. No authentication or TLS
  of its own; bind it to a private address.
- `mimir` role: Mimir with all components in one process, blocks on local disk
  or in an S3-compatible bucket, Prometheus remote write and OTLP in, the
  Prometheus query API out. Points its internal query path at the bind
  address, which Mimir otherwise dials on 127.0.0.1.
- `tempo` role: Tempo 3 in monolithic mode, OTLP in over gRPC and HTTP, blocks
  on local disk or in an S3-compatible bucket. Keeps every local path under
  its data directory and points the querier and the backend worker at the bind
  address; Tempo 3 otherwise writes to /var/tempo and dials 127.0.0.1.
- `grafana` role: Grafana with Loki, Mimir and Tempo provisioned as data
  sources, linked from logs to traces and back. Releases are unpacked side by
  side behind a `current` link. The admin password is kept at the declared
  value on every run. Plugin preinstall and auto-update are off: with them on,
  Grafana replaced its bundled Prometheus and Tempo plugins from grafana.com at
  startup and the data sources stopped working.
