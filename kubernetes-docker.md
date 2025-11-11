# Kubernetes / Docker

## Kubernetes

Privileged Pod: A pod with `securityContext.privileged: true` (or container-level) can do nearly anything on the node — can mount host filesystems, access host network, and potentially escape container isolation.

## Docker privilege escalation

- Containers running with `--privileged` or with mounted Docker socket allowing effective host control
- HostPath mounts exposing sensitive directories
- SUID binaries / dangerous capabilities
