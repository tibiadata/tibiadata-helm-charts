# tibiadata-api-go Helm chart

## Installation

To add the tibiadata helm repo, run:

```console
helm repo add tibiadata https://charts.tibiadata.com
helm repo update
```

To install a release named tibiadata-api-go, run:

```console
helm install tibiadata-api-go tibiadata/tibiadata-api-go
```

## Chart Values

```console
helm show values tibiadata/tibiadata-api-go
```

## Value Details

### image

Specify the container image to use for the deployment. `image.tag` For example, the following sets the image to the `ghcr.io/tibiadata/tibiadata-api-go` repo and the `v4.1.2` tag. The container pulls the image if not already present:

```yaml
image:
  repository: ghcr.io/tibiadata/tibiadata-api-go
  tag: v4.1.2
  pullPolicy: IfNotPresent
```

The chart also supports specifying an image based on digest value:

```yaml
image:
  repository: ghcr.io/tibiadata/tibiadata-api-go
  digest: sha256:a6f902768cb71c0a0b391ec167146cd75bc07f063ad84f9022d98f7459f301bb
  pullPolicy: IfNotPresent
```

If neither tag or digest is specified, the `appVersion` of the chart is used as a default.

### secret

Reference an existing Kubernetes `Secret` for any sensitive values the app needs (e.g. an
API token such as `TIBIA_FANSITEAPI_TOKEN`, credentials, etc.). This chart never creates or
manages the Secret's contents — create/populate it yourself out-of-band (`kubectl`, a GitOps
secret tool, external-secrets, etc.) before installing or upgrading the release with this
enabled. The Secret can hold as many keys as you like; it isn't tied to any specific value:

```console
kubectl create secret generic tibiadata-api-go-secrets \
  --from-literal=TIBIA_FANSITEAPI_TOKEN=xxxxxxxx
```

```yaml
secret:
  enabled: true
  name: tibiadata-api-go-secrets
```

By default all keys in the Secret are loaded as environment variables (via `envFrom`). To load
only specific keys, or to rename them, set `secret.env`:

```yaml
secret:
  enabled: true
  name: tibiadata-api-go-secrets
  env:
    - name: TIBIA_FANSITEAPI_TOKEN
      key: TIBIA_FANSITEAPI_TOKEN
```

#### Letting the chart create the Secret

If you'd rather not run a separate `kubectl create secret` before the first install, set
`secret.create: true`. The chart then creates the Secret itself, but only as an empty
placeholder (one blank-value key per entry in `secret.keys`) so the release installs cleanly
and the pod's `envFrom`/`secretKeyRef` references resolve:

```yaml
secret:
  enabled: true
  create: true
  name: tibiadata-api-go-secrets
  keys:
    - TIBIA_FANSITEAPI_TOKEN
```

After installing, populate the real value(s) yourself, e.g.:

```console
kubectl edit secret tibiadata-api-go-secrets
```

This is safe to leave enabled across upgrades: the template looks up the Secret's current data
first and re-emits it unchanged if it already exists, so `helm upgrade` never clobbers values
you've since populated. The Secret is also annotated with `helm.sh/resource-policy: keep`, so
it survives `helm uninstall` (and won't be deleted if you later set `secret.create: false`
again).

Note: `secret.create`'s existence check relies on Helm's `lookup` function, which only works
against a live cluster. It always renders as "not yet existing" (i.e. the empty-placeholder
branch) under `helm template` or `helm lint` run without a cluster context — this is expected
and does not affect real installs/upgrades.

Note: since the Secret's content is managed outside Helm, upgrading the release does not
detect changes to it and pods are not automatically restarted when you rotate the value. Use
a tool like [Reloader](https://github.com/stakater/Reloader) to watch the Secret, or trigger a
rollout yourself with `kubectl rollout restart deployment/<release-name>-tibiadata-api-go`.
