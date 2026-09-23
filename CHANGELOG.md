# Changelog

All notable changes to this collection are documented here.
This project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added

- `grafana_release` role: installs Loki, Mimir, Tempo or Alloy from its GitHub
  release. Reads both checksum formats Grafana Labs publishes and refuses to
  install an asset it finds no checksum for. Compares the installed version,
  so a version bump upgrades in place, and tells the calling role through
  `grafana_release_changed`, which `--check` reports too.
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
