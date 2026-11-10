---
title: "3.10"
weight: 310
---

<Alert severity="info">
  This guide covers migrating from Emissary 3.10.Z to Emissary 4.2. If
  this is not your <b>exact</b> situation, see the <a href="../../../migration-matrix/">migration
  matrix</a>.
</Alert>

<Alert severity="warning">
  This guide is written for upgrading an installation made by applying YAML manifests
  (<code>emissary-crds.yaml</code> and <code>emissary-emissaryns.yaml</code>).
  If you originally installed with Helm, see the <a href="../../helm/emissary-3.10/">Helm-based
  upgrade instructions</a>.
</Alert>

Emissary 4 does not publish YAML manifests. Instead, you render them from the Helm charts with
`helm template` and apply the output with `kubectl`, so you need the `helm` CLI (3.8 or later) but
do not install anything with Helm. Review the breaking changes for 4.0.1 in the
[CHANGELOG](https://github.com/emissary-ingress/emissary/blob/v4.2.0/CHANGELOG.md) before you start.

If you edited `emissary-emissaryns.yaml` before applying it, pass the same settings to
`helm template` with `--set` or `-f`.

## Before you start

Check which CRD versions are stored in your cluster:

```bash
kubectl get crd mappings.getambassador.io -o jsonpath='{.status.storedVersions}'
```

If the list includes `v2`, you must render the 4.2 CRDs with `enableLegacyVersions=true`, or
Kubernetes rejects the update. If the list also includes `v1`, add `enableV1=true` as well. The
commands below assume `v2` only.

<Alert severity="warning">
  Never delete the Emissary CRDs as part of an upgrade. Deleting a CRD deletes every resource of
  that kind, so all of your <code>Mapping</code>s, <code>Host</code>s and other Emissary resources would be lost.
</Alert>

## Migration Steps

1. **Upgrade the CRDs.**

   ```bash
   helm template emissary-crds oci://ghcr.io/emissary-ingress/emissary-crds-chart --version 4.2.0 \
     --namespace emissary-system \
     --set enableLegacyVersions=true \
     | kubectl apply -f -
   kubectl -n emissary-system rollout status deploy/emissary-apiext
   ```

   Upgrading the CRDs does not interrupt traffic. Your `getambassador.io/v2` resources continue to
   be served by the 4.2 `emissary-apiext` conversion webhook.

2. **Upgrade Emissary.**

   Render the chart with the release name `emissary-ingress`. That makes the generated resources
   match the names and selectors in `emissary-emissaryns.yaml`, so `kubectl apply` performs a rolling
   update of your existing Deployment. With any other release name you would get a second
   Deployment alongside the old one.

   ```bash
   helm template emissary-ingress oci://ghcr.io/emissary-ingress/emissary-ingress --version 4.2.0 \
     --namespace emissary \
     --set waitForApiext.enabled=true \
     | kubectl apply -f -
   kubectl -n emissary rollout status deploy/emissary-ingress
   ```

3. **Remove the leftover 3.10 resources.**

   Emissary 4.2 renames one Role and RoleBinding, so the 3.10 ones are no longer used:

   ```bash
   kubectl -n emissary-system delete role/emissary-ingress-apiext rolebinding/emissary-ingress-apiext
   ```

4. **Check the upgrade.**

   ```bash
   kubectl -n emissary get pods -o jsonpath='{.items[*].spec.containers[*].image}'
   kubectl get mappings.v2.getambassador.io -A
   ```

   The Emissary Pods should run `ghcr.io/emissary-ingress/emissary:4.2.0`.
