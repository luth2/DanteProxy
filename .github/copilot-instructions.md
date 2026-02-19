# Copilot Instructions — DanteProxy

## Big picture
- This repo has two deliverables: a Dante SOCKS5 container in `Container/` and a Helm chart in `helm-dante-proxy/` for deploying that container to Kubernetes.
- Runtime config is externalized: the container expects `/etc/danted/danted.conf` and uses a non-root user (`dante`, uid/gid `10001`) with a writable `/var/run/danted` directory.

## Key files and conventions
- `Container/DOCKERFILE` builds a minimal Ubuntu-based image and copies `danted` from a build stage; keep the non-root user and directory permissions aligned with Kubernetes security context defaults.
- `Container/dante-bsp.conf` is the sample Dante config; its `logoutput: stdout` and `internal: 0.0.0.0 port = 1080` match the container health checks and ports.
- `helm-dante-proxy/templates/deployment.yaml` is based on the Helm starter chart; it currently assumes an HTTP port named `http` and uses values-driven probes.
- `helm-dante-proxy/values.yaml` ships with default `nginx` image settings; set `image.repository`, `image.tag`, and `service.port` to match the Dante image and port (1080) when deploying.

## Workflows from docs
- Local container run is documented in `README.md` using an image name `dante:prod` and a bind-mounted config to `/etc/danted/danted.conf`.
- Kubernetes deployment guidance in `README.md` uses a ConfigMap-mounted `/etc/danted` and an `emptyDir` for `/var/run/danted` plus tight `securityContext` settings; keep these in sync with the Dockerfile’s non-root user.

## Integration points
- The container is a thin wrapper around the `danted` binary from the distro package; changes to the proxy behavior should be made in the Dante config mounted at `/etc/danted/danted.conf`.
- Helm chart templates use standard `_helpers.tpl` naming/label helpers; keep naming consistent with chart name or `fullnameOverride`.

## Agent focus areas
- When updating container behavior, edit `Container/dante-bsp.conf` and validate any required runtime dirs/permissions in `Container/DOCKERFILE`.
- When updating Kubernetes deployment, adjust both `helm-dante-proxy/values.yaml` and `templates/*.yaml` so ports, probes, and volume mounts reflect the Dante SOCKS service (TCP 1080), not HTTP defaults.
