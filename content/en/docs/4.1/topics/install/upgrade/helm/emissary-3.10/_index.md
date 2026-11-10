---
title: "3.10"
weight: 310
---

<Alert severity="info">
  This guide covers migrating from Emissary 3.10.Z to Emissary 4.1. If
  this is not your <b>exact</b> situation, see the <a href="../../../migration-matrix/">migration
  matrix</a>.
</Alert>

<Alert severity="warning">
  This guide is written for upgrading an installation originally made using Helm.
  If you did not install with Helm, see the <a href="../../yaml/emissary-3.10/">YAML-based
  upgrade instructions</a>.
</Alert>

Emissary 4 publishes its Helm charts only to the GitHub Container Registry, as two charts:
`oci://ghcr.io/emissary-ingress/emissary-crds-chart` for the CRDs and
`oci://ghcr.io/emissary-ingress/emissary-ingress` for Emissary itself. The chart version now matches
the Emissary version. Review the breaking changes for 4.0.1 in the
[CHANGELOG](https://github.com/emissary-ingress/emissary/blob/v4.1.0/CHANGELOG.md) before you start.

These steps use the release names and namespaces from the 3.10 quick start: CRDs in the
`emissary-crds` release in `emissary-system`, and Emissary in the `emissary` release in `emissary`.
Adjust them if yours differ.

## Before you start

1. **Check which CRD versions are stored in your cluster.**

   ```bash
   kubectl get crd mappings.getambassador.io -o jsonpath='{.status.storedVersions}'
   ```

   If the list includes `v2`, you must install the 4.1 CRDs with `enableLegacyVersions=true`. Without
   it, the upgrade fails with `status.storedVersions[0]: Invalid value: "v2": must appear in
   spec.versions` and nothing is changed. If the list also includes `v1`, add `enableV1=true` as well.
   The commands below assume `v2` only.

2. **Check how your CRDs were installed.**

   ```bash
   helm list -n emissary-system
   ```

   If an `emissary-crds` release is listed, use step 1a below. If not, your CRDs were applied with
   `kubectl apply`, so use step 1b.

<Alert severity="warning">
  Never delete the Emissary CRDs as part of an upgrade. Deleting a CRD deletes every resource of
  that kind, so all of your <code>Mapping</code>s, <code>Host</code>s and other Emissary resources would be lost.
</Alert>

## Migration Steps

1. **Upgrade the CRDs.**

   **1a. CRDs installed with Helm:**

   ```bash
   helm upgrade emissary-crds -n emissary-system \
     oci://ghcr.io/emissary-ingress/emissary-crds-chart --version 4.1.0 \
     --set enableLegacyVersions=true \
     --wait
   ```

   **1b. CRDs installed with `kubectl apply`:** Helm will not take over resources it did not
   create, so first mark the existing CRDs and `emissary-apiext` resources as belonging to the
   `emissary-crds` release, then install it:

   ```bash
   CLUSTER_OBJS="$(kubectl get crd -o name | grep '\.getambassador\.io$') clusterrole/emissary-apiext clusterrolebinding/emissary-apiext"
   NS_OBJS="serviceaccount/emissary-apiext role/emissary-apiext rolebinding/emissary-apiext service/emissary-apiext deployment/emissary-apiext"

   kubectl label --overwrite $CLUSTER_OBJS app.kubernetes.io/managed-by=Helm
   kubectl annotate --overwrite $CLUSTER_OBJS \
     meta.helm.sh/release-name=emissary-crds meta.helm.sh/release-namespace=emissary-system
   kubectl -n emissary-system label --overwrite $NS_OBJS app.kubernetes.io/managed-by=Helm
   kubectl -n emissary-system annotate --overwrite $NS_OBJS \
     meta.helm.sh/release-name=emissary-crds meta.helm.sh/release-namespace=emissary-system

   helm install emissary-crds -n emissary-system \
     oci://ghcr.io/emissary-ingress/emissary-crds-chart --version 4.1.0 \
     --set enableLegacyVersions=true \
     --wait
   ```

   Upgrading the CRDs does not interrupt traffic. Your `getambassador.io/v2` resources continue to
   be served by the 4.1 `emissary-apiext` conversion webhook.

2. **Upgrade Emissary.**

   ```bash
   helm upgrade emissary -n emissary \
     oci://ghcr.io/emissary-ingress/emissary-ingress --version 4.1.0 \
     --reset-values -f your-values.yaml \
     --set waitForApiext.enabled=true \
     --wait
   ```

   Use `--reset-values` and pass your own settings with `-f` (or `--set`). Without it, Helm keeps
   the values stored from your 3.x release, including any pinned `image.tag`, and Emissary stays on
   3.10.

   <Alert severity="warning">
     The 4.x chart names its resources differently. For a release named <code>emissary</code>, the
     <code>emissary-emissary-ingress</code> Deployment and Service are replaced by
     <code>emissary</code> and <code>emissary-admin</code>. This means your LoadBalancer gets a new
     address, and traffic stops until the new Pods are ready (about 40 seconds in testing). To avoid
     both, follow <a href="#keeping-the-existing-service">Keeping the existing Service</a> instead of
     this step.
   </Alert>

3. **Check the upgrade.**

   ```bash
   kubectl -n emissary get deploy,svc
   kubectl -n emissary get pods -o jsonpath='{.items[*].spec.containers[*].image}'
   kubectl get mappings.v2.getambassador.io -A
   ```

   The Emissary Pods should run `ghcr.io/emissary-ingress/emissary:4.1.0`.

## Keeping the existing Service

This replaces step 2 above. It keeps the old Service, and its LoadBalancer address, serving traffic
throughout the upgrade, then points it at the 4.1 Pods.

1. Tell Helm to keep the old Deployment and Service when it upgrades the release:

   ```bash
   kubectl -n emissary annotate deploy/emissary-emissary-ingress svc/emissary-emissary-ingress \
     helm.sh/resource-policy=keep
   ```

2. Upgrade without `--wait`. Helm's `--wait` also waits for the new `emissary` LoadBalancer Service
   to get an address, which may not happen while the old Service still holds the ports.

   ```bash
   helm upgrade emissary -n emissary \
     oci://ghcr.io/emissary-ingress/emissary-ingress --version 4.1.0 \
     --reset-values -f your-values.yaml \
     --set waitForApiext.enabled=true
   kubectl -n emissary rollout status deploy/emissary
   ```

   The 3.10 Pods keep serving traffic until you remove them in the next step.

3. Point the old Service at the 4.1 Pods, then remove the old Deployment:

   ```bash
   kubectl -n emissary patch svc emissary-emissary-ingress --type merge \
     -p '{"spec":{"selector":{"app.kubernetes.io/name":"emissary"}}}'
   kubectl -n emissary delete deploy emissary-emissary-ingress
   ```

4. Once your DNS and clients use the new `emissary` Service, delete the old one:

   ```bash
   kubectl -n emissary delete svc emissary-emissary-ingress
   ```
