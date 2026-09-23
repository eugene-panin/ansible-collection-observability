# mimir

Runs Mimir as a single process, every component in one, storing blocks on
local disk or in an S3-compatible bucket. It takes Prometheus remote write and
OTLP over HTTP, and answers the Prometheus query API under `/prometheus`.

```yaml
- name: Mimir
  hosts: observability
  become: true
  roles:
    - role: eugene_panin.observability.mimir
      vars:
        mimir_bind_address: 10.0.0.10
        mimir_storage: s3
        mimir_s3_endpoint: s3.eu-west-1.amazonaws.com
        mimir_s3_bucket: acme-mimir
        mimir_s3_region: eu-west-1
        mimir_s3_access_key: "{{ vault_mimir_s3_access_key }}"
        mimir_s3_secret_key: "{{ vault_mimir_s3_secret_key }}"
```

## Storage

`mimir_storage: filesystem` keeps the object storage under
`mimir_data_dir/data`. `mimir_storage: s3` puts blocks, rules and alertmanager
state in one bucket under `blocks/`, `ruler/` and `alertmanager/`; the TSDB
head and write-ahead log stay on the host. The bucket must exist.

Both modes are tested the same way: a sample is pushed over OTLP, the head is
flushed so the block is shipped, Mimir is restarted and the sample is queried
back through the Prometheus API. The S3 test also finds the block's
`meta.json` in the bucket and nothing in local object storage.

## Addresses

Everything binds to `mimir_bind_address`, including gRPC. The components that
talk to each other inside the process are pointed at that address too; left
to its defaults, Mimir's query path dials `127.0.0.1` and fails when gRPC is
not listening there.

## What it does not do

- No authentication and no TLS. Bind it to a private address, or put a proxy
  in front.
- One node. A cluster needs a shared ring and object storage, and there is no
  test for it yet.
- Multi-tenancy is off; everything lands in the tenant `anonymous`.

## Variables

| Variable | Default | Purpose |
|---|---|---|
| `mimir_bind_address` | required | Address everything binds to |
| `mimir_version` | `3.2.1` | Exact version |
| `mimir_storage` | `filesystem` | `filesystem` or `s3` |
| `mimir_s3_endpoint`, `mimir_s3_bucket` | `""` | Required for `s3` |
| `mimir_s3_access_key`, `mimir_s3_secret_key` | `""` | Credentials, from a vault |
| `mimir_retention_period` | `0s` | How long blocks are kept; `0s` is forever |
| `mimir_http_port`, `mimir_grpc_port` | `9009`, `9095` | Ports |
| `mimir_extra_config` | `{}` | Merged into the rendered configuration |

## Notes

- Changing `mimir_version` replaces the binary and restarts Mimir; `--check`
  reports both.
- Mimir writes an activity log it reads back on start. The role puts it in
  `mimir_data_dir`; by default Mimir writes it to the working directory, which
  for a service is `/`.
