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

Reference an existing Kubernetes `Secret` for sensitive values (e.g. `TIBIA_FANSITEAPI_TOKEN`).
This chart never creates or manages the Secret's contents — create/populate it yourself
out-of-band (`kubectl`, a GitOps secret tool, external-secrets, etc.) before installing or
upgrading the release with this enabled:

```console
kubectl create secret generic tibiadata-api-go-fansite-token \
  --from-literal=TIBIA_FANSITEAPI_TOKEN=xxxxxxxx
```

```yaml
secret:
  enabled: true
  name: tibiadata-api-go-fansite-token
```

By default all keys in the Secret are loaded as environment variables (via `envFrom`). To load
only specific keys, or to rename them, set `secret.env`:

```yaml
secret:
  enabled: true
  name: tibiadata-api-go-fansite-token
  env:
    - name: TIBIA_FANSITEAPI_TOKEN
      key: TIBIA_FANSITEAPI_TOKEN
```

Note: since the Secret's content is managed outside Helm, upgrading the release does not
detect changes to it and pods are not automatically restarted when you rotate the value. Use
a tool like [Reloader](https://github.com/stakater/Reloader) to watch the Secret, or trigger a
rollout yourself with `kubectl rollout restart deployment/<release-name>-tibiadata-api-go`.
