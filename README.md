# Meru Kyverno

The trust rules of **Meru**, a small in-house container platform (CaaS) on k3s: what each app's namespace may run,
enforced by [Kyverno](https://kyverno.io) when a pod is created. Argo CD applies this repo to the cluster's
`kyverno` namespace. Changes go in through pull requests that the platform owner merges - this repo decides what the
cluster trusts, so nothing here merges on its own.

## What's here

- `keys/meru-signing-keys.yaml` - one line per app: its namespace and the cosign public key its images must be
  signed with. Adding an app's line is trusting that app's builds. The private halves live in OpenBao, where only
  that app's build jobs can fetch them ([Meru/sindri](https://git.clockmost.com/Meru/sindri)).
- `signatures/meru-signed.yaml` - one rule for every app namespace: every image a pod runs must carry a signature
  from the key listed for that namespace. A namespace with no line runs nothing, and says so. Build namespaces
  (`<app>-build`) are left out: their runner and BuildKit are upstream images.
- `exceptions/<app>-build.yaml` - one per app: its build namespace may be Pod Security "baseline" instead of
  "restricted", because rootless BuildKit needs it. It names exactly one namespace and goes when the app goes.

A stolen key of one app can only sign images that run in that app's namespace.

`cluster/` holds the rules for every workload namespace, applied by their own Argo CD Application
(`kyverno-policies` in [Meru/meru](https://git.clockmost.com/Meru/meru)), not by this repo's root:

- images by digest only, from the allowed registries (our [Zot](https://git.clockmost.com/Meru/zot), which mirrors
  the public ones);
- Pod Security "restricted" namespaces, resource requests and limits, health probes, and Meru's ownership labels on
  every workload;
- `cluster/exceptions/` - the platform parts that may break one of these, each with its reason (Falco runs
  privileged).

## Adding an app

1. Its line in `keys/meru-signing-keys.yaml` (the public key from its build setup).
2. `exceptions/<app>-build.yaml` (copy an existing one, change the name and namespace).
3. Both files in `kustomization.yaml`.

## License

MIT - see `LICENSE`.
