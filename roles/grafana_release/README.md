# grafana_release

Installs one Grafana Labs binary from its GitHub release: Loki, Mimir, Tempo
or Alloy. The `loki`, `mimir`, `tempo` and `alloy` roles call it.

```yaml
- ansible.builtin.include_role:
    name: eugene_panin.observability.grafana_release
  vars:
    grafana_release_product: loki
    grafana_release_version: 3.7.8
```

## What it checks

Grafana Labs publishes checksums two ways. Loki, Tempo and Alloy ship one
`SHA256SUMS` file per release. Mimir ships a file per asset holding nothing but
the hash. The role reads either, picks the hash for the exact asset it is about
to download, and stops with an error naming the asset when there is none. The
download is then checked against that hash.

The installed version is read from `<product> --version`, so changing
`grafana_release_version` replaces the binary in place, and a host that already
has it costs one command per run.

The checksums are trusted because they come from GitHub over TLS. If you point
`grafana_release_base_url` at a mirror, the trust moves to the mirror.

## Variables

| Variable | Default | Purpose |
|---|---|---|
| `grafana_release_product` | required | `loki`, `mimir`, `tempo` or `alloy` |
| `grafana_release_version` | required | Exact version, no leading `v` |
| `grafana_release_bin_dir` | `/usr/local/bin` | Where the binary goes |
| `grafana_release_download_dir` | `/var/cache/grafana-releases` | Downloads and unpacked archives |
| `grafana_release_base_url` | `https://github.com` | Mirror of the `grafana/<product>` release pages |

Architectures: `x86_64` and `aarch64`. `unzip` must be on the host.

## What the calling role gets back

`grafana_release_changed` is true when the installed version differs from the
wanted one. It is set before anything is downloaded; restart the service on
it.
