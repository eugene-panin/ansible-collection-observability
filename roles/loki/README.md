# loki

Runs Loki as a single process, every component in one, storing chunks and
index either on local disk or in an S3-compatible bucket.

```yaml
- name: Loki
  hosts: observability
  become: true
  roles:
    - role: eugene_panin.observability.loki
      vars:
        loki_bind_address: 10.0.0.10
        loki_storage: s3
        loki_s3_endpoint: s3.eu-west-1.amazonaws.com
        loki_s3_bucket: acme-loki
        loki_s3_region: eu-west-1
        loki_s3_access_key: "{{ vault_loki_s3_access_key }}"
        loki_s3_secret_key: "{{ vault_loki_s3_secret_key }}"
```

## Storage

`loki_storage: filesystem` keeps everything under `loki_data_dir`. Simple, and
the data lives and dies with the host.

`loki_storage: s3` sends chunks and the TSDB index to the bucket; only the
write-ahead log and working files stay on the host. Rebuilding the host loses
at most what had not been flushed yet. The bucket must exist; Loki does not
create it.

Both modes are tested the same way: a line is pushed, the in-memory chunks are
flushed, Loki is restarted and the line is read back. The S3 test also checks
the chunks and the index are in the bucket and nothing went to local chunk
storage.

## What it does not do

- No authentication and no TLS. Loki has neither built in for its HTTP API.
  Bind it to a private address, or put a proxy in front that does both.
- One node. Running several needs a shared ring and object storage, and there
  is no test for it yet.
- `auth_enabled` is off, so everything lands in the single tenant `fake`. Turn
  it on with `loki_auth_enabled` and send `X-Scope-OrgID` if you need tenants.

## Variables

| Variable | Default | Purpose |
|---|---|---|
| `loki_bind_address` | required | Address the HTTP and gRPC servers bind to |
| `loki_version` | `3.7.8` | Exact version |
| `loki_storage` | `filesystem` | `filesystem` or `s3` |
| `loki_s3_endpoint`, `loki_s3_bucket` | `""` | Required for `s3` |
| `loki_s3_access_key`, `loki_s3_secret_key` | `""` | Credentials, from a vault |
| `loki_s3_region` | `us-east-1` | Bucket region |
| `loki_s3_insecure` | `false` | Plain HTTP to the endpoint |
| `loki_retention_period` | `744h` | How long logs are kept |
| `loki_http_port`, `loki_grpc_port` | `3100`, `9096` | Ports |
| `loki_extra_config` | `{}` | Merged into the rendered configuration |

## Notes

- Changing `loki_version` replaces the binary and restarts Loki; `--check`
  reports both.
- `loki_schema_from` is the first day of the schema. Leave it alone once data
  exists; a new schema is added, never edited.
