# Guest images

A small collection of optional base images that bundle a real init system (systemd, OpenRC) for use with [microsandbox](https://github.com/superradcompany/microsandbox).

You don't need these to run microsandbox. Its default agent-as-PID-1 is enough for one-shot processes and most workloads. Reach for these when you specifically need `systemctl`, `loginctl`, dbus, or a workload that ships only as a `.deb` / `.rpm` with a service unit.

| Image | Base | Init | Primary tag | Pull |
|-------|------|------|-------------|------|
| [`debian-systemd`](./debian-systemd) | `debian:bookworm` | systemd | `:12`, `:bookworm` | `ghcr.io/superradcompany/debian-systemd:12` |
| [`ubuntu-systemd`](./ubuntu-systemd) | `ubuntu:24.04` | systemd | `:24.04`, `:noble` | `ghcr.io/superradcompany/ubuntu-systemd:24.04` |
| [`fedora-systemd`](./fedora-systemd) | `fedora:40` | systemd | `:40` | `ghcr.io/superradcompany/fedora-systemd:40` |
| [`alpine-openrc`](./alpine-openrc) | `alpine:3.20` | OpenRC + BusyBox | `:3.20` | `ghcr.io/superradcompany/alpine-openrc:3.20` |

All images are published as `linux/amd64` + `linux/arm64` multi-arch manifests on `ghcr.io/superradcompany/<name>:<tag>`.

## Use

```bash
msb run ghcr.io/superradcompany/debian-systemd:12 \
  --memory 1G --cpus 2 \
  --init auto \
  -- bash
```

See the [Custom init system](https://microsandbox.dev/sandboxes/customize#custom-init-system) docs and the [Booting Under systemd](https://microsandbox.dev/recipes/systemd-services) recipe for the full mechanic.

## Tags

Each image carries four kinds of tags:

- **Primary** (`:<distro-version>`, e.g. `debian-systemd:12`): the canonical pin most callers use. Mutable, rebuilt weekly.
- **Distro alias** (`:<codename>`, e.g. `:bookworm`): mirror of the primary, friendlier for humans. Mutable.
- **Latest** (`:latest`): newest supported distro version. Mutable.
- **Dated** (`:<distro-version>-YYYY-MM-DD`, e.g. `:12-2026-05-12`): immutable digest pin for reproducible builds.

For reproducible builds, prefer a dated tag or a digest (`@sha256:…`).

## Rebuild cadence

Rebuilt weekly (Monday 06:00 UTC) so each image picks up its upstream base's security patches without us having to do anything. The same Dockerfiles drive every rebuild, and each rebuild is gated by a smoke test that runs the produced image and verifies an init binary exists at one of the paths `--init auto` checks. Failed rebuilds leave the previous `:latest` digest in place rather than poisoning the tag.

## Adding a new image

1. Add a directory `<name>/` at the repo root with `Dockerfile` and `README.md`.
2. Add a matrix entry in `.github/workflows/build.yml` (both the `build` and `manifest` jobs).
3. Open a PR. CI builds and smoke-tests every image on every PR; pushes to GHCR only happen on the weekly rebuild.
