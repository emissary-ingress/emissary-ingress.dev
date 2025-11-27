---
title: 3.9
---

<Alert severity="info">
  This guide covers migrating from Emissary 3.9.Z to Emissary 3.10. If
  this is not your <b>exact</b> situation, see the <a href="../../../../migration-matrix">migration
  matrix</a>.
</Alert>

<Alert severity="warning">
  This guide is written for upgrading an installation made without using Helm.
  If you originally installed with Helm, see the <a href="../../../helm/emissary-3.8/emissary-3.X">Helm-based
  upgrade instructions</a>.
</Alert>

Since Emissary's configuration is entirely stored in Kubernetes resources, upgrading
between versions is straightforward.

### Resources to check before migrating to 3.10.

## Migration Steps

Migration is a two-step process:

1. **Install new CRDs.**

   Before installing Emissary 3.10 itself, you need to update the CRDs in
   your cluster. This is mandatory during any upgrade of Emissary.

   ```bash
   kubectl apply -f https://app.getambassador.io/yaml/emissary/3.10/emissary-crds.yaml
   kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system
   ```

   <Alert severity="info">
     Emissary 3.10 includes a Deployment in the `emissary-system` namespace
     called <code>emissary-apiext</code>. This is the APIserver extension
     that supports converting Emissary CRDs between <code>getambassador.io/v2</code>
     and <code>getambassador.io/v3alpha1</code>. This Deployment needs to be running at
     all times.
   </Alert>

   <Alert severity="warning">
     If the <code>emissary-apiext</code> Deployment's Pods all stop running,
     you will not be able to use <code>getambassador.io/v3alpha1</code> CRDs until restarting
     the <code>emissary-apiext</code> Deployment.
   </Alert>

   <Alert severity="warning">
    There is a known issue with the <code>emissary-apiext</code> service that impacts all Emissary 2.x and 3.x users. Specifically, the TLS certificate used by apiext expires one year after creation and does not auto-renew. All users who are running Emissary/Ambassador Edge Stack 2.x or 3.x with the apiext service should proactively renew their certificate as soon as practical by running <code>kubectl delete --all secrets --namespace=emissary-system</code> to delete the existing certificate, and then restart the <code>emissary-apiext</code> deployment with <code>kubectl rollout restart deploy/emissary-apiext -n emissary-system</code>.
    This will create a new certificate with a one year expiration. We will issue a software patch to address this issue well before the one year expiration. Note that certificate renewal will not cause any downtime.
   </Alert>

2. **Install Emissary 3.10.**

   After installing the new CRDs, upgrade Emissary 3.10.

   <Alert severity="info">
     Our <a href="https://app.getambassador.io/yaml/emissary/3.10/emissary-emissaryns.yaml"><code>emissary-emissaryns.yaml</code></a> file
     uses the `emissary` namespace, since this is the default for Emissary.
     We also publish <a href="https://app.getambassador.io/yaml/emissary/3.10/emissary-defaultns.yaml"><code>emissary-defaultns.yaml</code></a> for the
     `default` namespace. For any other namespace, you should download one of these files and edit the namespaces manually.
   </Alert>

   ```bash
   kubectl apply -f https://app.getambassador.io/yaml/emissary/3.10/emissary-emissaryns.yaml && \
   kubectl rollout status  -n emissary deployment/emissary-ingress -w
   ```
