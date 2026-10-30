# Policy exceptions

One Kyverno `PolicyException` (`policies.kyverno.io`) per workload that can't meet a rule, in namespace `kyverno`, with the reason in its
`meru.clockmost.com/reason` annotation. Add the file to `../kustomization.yaml`.

| file | rule skipped | for |
|------|--------------|-----|
| `cert-manager.yaml` | 6, probes | cert-manager's controller (liveness only) and cainjector (no health endpoint) |
| `falco.yaml` | 4, restricted namespaces | the `falco` namespace (privileged, eBPF + runtime sockets) |

Apps' build exceptions are not here: each app's is `../../exceptions/<app>-build.yaml`, next to its signing key.
