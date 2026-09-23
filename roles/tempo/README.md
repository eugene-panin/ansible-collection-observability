# tempo

Runs Tempo 3 in monolithic mode: one process, no Kafka, OTLP in over gRPC and
HTTP, trace blocks on local disk or in an S3-compatible bucket.

```yaml
- name: Tempo
  hosts: observability
  become: true
  roles:
    - role: eugene_panin.observability.tempo
      vars:
        tempo_bind_address: 10.0.0.10
        tempo_storage: s3
        tempo_s3_endpoint: s3.eu-west-1.amazonaws.com
        tempo_s3_bucket: acme-tempo
        tempo_s3_region: eu-west-1
        tempo_s3_access_key: "{{ vault_tempo_s3_access_key }}"
        tempo_s3_secret_key: "{{ vault_tempo_s3_secret_key }}"
```

## Storage

Recent traces live in the live store and its write-ahead log under
`tempo_data_dir`. Completed blocks go to `tempo_data_dir/blocks`, or to the
bucket with `tempo_storage: s3`. The bucket must exist.

Both modes are tested the same way: a span is sent over OTLP, the test waits
until the trace can be read with `mode=blocks`, which reads nothing but block
storage, then Tempo is restarted and the trace is read from block storage
again. The S3 test also finds the block's `meta.json` in the bucket.

## Defaults Tempo 3 gets wrong on such a host

Out of the box, Tempo 3 in monolithic mode assumes it may write to
`/var/tempo` and that its gRPC server listens on `127.0.0.1`. Neither holds for
a service running as its own user and bound to one address, and the failures
are quiet: the live store fails and takes ingestion with it, and queries hang.
The role sets:

- every local path, including the live store's WAL, its shutdown marker and
  the backend scheduler's work cache, under `tempo_data_dir`;
- the querier's frontend address and the backend worker's scheduler address
  to the bind address.

## What it does not do

- No authentication and no TLS. Bind it to a private address.
- No metrics generator. Span metrics and service graphs need a Prometheus
  remote write target, and there is no test for that yet.
- One node.

## Variables

| Variable | Default | Purpose |
|---|---|---|
| `tempo_bind_address` | required | Address everything binds to |
| `tempo_version` | `3.0.3` | Exact version |
| `tempo_storage` | `filesystem` | `filesystem` or `s3` |
| `tempo_s3_endpoint`, `tempo_s3_bucket` | `""` | Required for `s3` |
| `tempo_s3_access_key`, `tempo_s3_secret_key` | `""` | Credentials, from a vault |
| `tempo_retention_period` | `336h` | How long blocks are kept |
| `tempo_otlp_grpc_port`, `tempo_otlp_http_port` | `4317`, `4318` | OTLP receivers |
| `tempo_http_port`, `tempo_grpc_port` | `3200`, `9097` | Query API and internal gRPC |
| `tempo_extra_config` | `{}` | Merged into the rendered configuration |

## Notes

- Changing `tempo_version` replaces the binary and restarts Tempo.
