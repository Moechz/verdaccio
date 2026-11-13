# verdaccio-for-tos

> TerraMaster TOS 7 App Center — **Docker application** package for
> [Verdaccio](https://github.com/verdaccio/verdaccio).

## What this is

A four-file TOS Docker application archive (`verdacciodocker.tar.gz`) that installs
Verdaccio on TOS 7 through the App Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id `verdacciodocker`)
- `verdacciodocker.lang` — 23-language superset (TOS requires 14)
- `verdacciodocker.svg` — application icon (469 bytes, 3 SVG elements)
- `docker-compose.yml` — one service, image `verdaccio/verdaccio:7.1.0`

## How it works

| | |
|---|---|
| Container image | `verdaccio/verdaccio:7.1.0` (Docker Hub, used unmodified) |
| Open | `http://<NAS-IP>:8007/` (container port 4873) |
| Runs as | non-root, `user: "1000:1000"` |
| Persistent data | `/verdaccio/storage` → `/Volume1/DockerAppData/verdacciodocker/storage` |
| Logs | `docker logs verdacciodocker` (stdout/stderr) |

`/verdaccio/storage` (mounted from `DockerAppData/verdacciodocker/storage`) keeps the published tarballs and the package metadata (`.verdaccio-db.json`, per-package folders).

## Design highlights

- **Non-root.** `user: "1000:1000"`; verified on real hardware (the container reports uid 1000).
- **Fixed image tag, never `:latest`**, Docker Hub only, no `privileged`, no `network_mode: host`, and no Docker socket, no `cap_add`.
- **Health check** probing the service on container port 4873.
- **Deterministic build**: GNU tar with uid/gid 0 and zeroed timestamps, gzip without a timestamp, plus the review-standards self-check above.
- **No database.** Verdaccio stores packages as files plus a small JSON index, so a backup is a folder copy.
- **One port for UI and registry.** The web UI and the npm registry share port 4873.

## Runtime file manifest

Everything the application writes on the NAS lives inside its own data
directory; the container-internal `/tmp` is discarded with the container. The
table lists what is created and how it behaves. Nothing else is written.

| Path | Purpose | Created when | Growth bound | Lifecycle |
|---|---|---|---|---|
| `/Volume1/DockerAppData/verdacciodocker/storage` | Verdaccio application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |

## Build

```bash
scripts/build.sh                  # -> out/verdacciodocker.tar.gz + .sha256
scripts/build.sh 7.1.0-2   # packaging iteration bump
scripts/build.sh 7.1.0-1 aarch64
```

The build stages the four files, substitutes the version/platform placeholders,
removes CRLF/BOM/AppleDouble, then runs
`tools/verify.py`, a review-standards self-check (archive layout, JSON validity,
required fields, 14+ language coverage, version consistency, Docker-Hub-only
images, fixed tag, reserved/recommended host ports, non-root user, per-service
healthcheck/TZ/restart, data path below the application data root, never the
`/Volume*` wildcard, comment-free sequence blocks, `x-app-meta` position, no
literal secrets and the SVG size/element limits).

## Local verification

```bash
./TOSAppSelfTestingTool -c out/verdacciodocker.tar.gz   # official spec check
bash test/install-and-verify.sh                  # on-device E2E (needs root)
```

> Docker applications cannot be side-loaded through the App Center's manual
> install page (it accepts `.deb` only); the official self-testing tool's `-i`
> mode or `docker compose` are the only ways to install one for testing.

## Store submission checklist

1. Public GitHub repo (code + README only).
2. Release **tag `v7.1.0-1`**, assets `verdacciodocker.tar.gz` +
   `verdacciodocker.tar.gz.sha256` (Docker asset naming: `<app_id>.tar.gz`, **no
   platform suffix**).
3. Developer platform → Add Application: ID `verdacciodocker`, package type Docker,
   repository URL, architecture `x86_64`.
4. Version Management → Add Version `7.1.0-1` → automated validation → review.

## License / branding

Verdaccio is distributed by its upstream authors under **MIT**. The container image is the official image from Docker Hub (`verdaccio/verdaccio:7.1.0`) and is used unmodified; no upstream source code is redistributed here. This packaging is an independent community submission by Moechz and is not affiliated with the upstream project or TerraMaster.

## Limitations

- The UI/registry is plain HTTP; use a TLS reverse proxy before exposing it to the internet.
- Publishing requires an account (`npm adduser`); the package does not create one for you.
- Large private packages grow the storage folder; check free space on the NAS when caching many public packages.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.
