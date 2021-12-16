import Alert from '@material-ui/lab/Alert';

# Upgrade $productName$ $version$ to $AESproductName$ $version$ (Helm)

<Alert severity="info">
  This guide covers migrating from $productName$ $version$ to $AESproductName$ $version$. If
  this is not your <b>exact</b> situation, see the <a href="../../../../migration-matrix">migration
  matrix</a>.
</Alert>

<Alert severity="warning">
  This guide is written for upgrading an installation originally made using Helm.
  If you did not install with Helm, see the <a href="../../../yaml/emissary-2.1/edge-stack-2.1">YAML-based
  upgrade instructions</a>.
</Alert>

You can upgrade from $productName$ to $AESproductName$ with a few simple commands. When you upgrade to $AESproductName$, you'll be able to access additional capabilities such as **automatic HTTPS/TLS termination, Swagger/OpenAPI support, API catalog, Single Sign-On, and more.** For more about the differences between $AESproductName$ and $OSSproductName$, see the [Editions page](/editions).

## Migration Overview

<Alert severity="warning">
  <b>Read the migration instructions below</b> before making any changes to your
  cluster!
</Alert>

The recommended strategy for migration is to run $productName$ $version$ and $AESproductName$
$version$ side-by-side in the same cluster. This gives $AESproductName$ $version$
and $AESproductName$ $version$ access to all the same configuration resources, with some
important notes:

1. **If needed, you can use labels to further isolate configurations.**

   If you need to prevent your $AESproductName$ $version$ installation from
   seeing a particular bit of $productName$ $version$ configuration, you can apply
   a Kubernetes label to the configuration resources that should be seen by
   your $AESproductName$ $version$ installation, then set its
   `AMBASSADOR_LABEL_SELECTOR` environment variable to restrict its configuration
   to only the labelled resources.

   For example, you could apply a `version-two: true` label to all resources
   that should be visible to $AESproductName$ $version$, then set
   `AMBASSADOR_LABEL_SELECTOR=version-two=true` in its Deployment.

2. **$AESproductName$ ACME and `Filter`s will be disabled while $productName$ is still running.**

   Since $AESproductName$ and $productName$ share configuration, $AESproductName$ cannot
   configure its ACME or other filter processors without also affecting $productName$. This
   migration process is written to simply disable these $AESproductName$ features to make
   it simpler to roll back, if needed. Alternate, you can isolate the two configurations
   as described above.

You can also migrate by [installing $AESproductName$ $version$ in a separate cluster](../migrate-to-2-alternate).
This permits absolute certainty that your $productName$ $version$ configuration will not be
affected by changes meant for $AESproductName$ $version$, but it is more effort.

## Side-by-Side Migration Steps

Migration is a five-step process:

1. **Install new CRDs.**

   Before installing $AESproductName$ $version$ itself, you need to update the CRDs in
   your cluster. This will allow supporting `getambassador.io/v2` resources as well as
   `getambassador.io/v3alpha1`; it is mandatory.

   ```
   kubectl apply -f https://app.getambassador.io/yaml/edge-stack/$version$/aes-crds.yaml && \
   kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system 
   ```

   <Alert severity="info">
     $AESproductName$ $version$ includes a Deployment in the `emissary-system` namespace
     called <code>$productDeploymentName$-apiext</code>. This is the APIserver extension
     that supports converting $productName$ CRDs between <code>getambassador.io/v2</code>
     and <code>getambassador.io/v3alpha1</code>. This Deployment needs to be running at
     all times.
   </Alert>

   <Alert severity="warning">
     If the <code>$productDeploymentName$-apiext</code> Deployment's Pods all stop running,
     you will not be able to use <code>getambassador.io/v3alpha1</code> CRDs until restarting
     the <code>$productDeploymentName$-apiext</code> Deployment.
   </Alert>

2. **Install $AESproductName$ $version$.**

   After installing the new CRDs, you need to install $AESproductName$ $version$ itself
   **in the same namespace as your existing $productName$ $version$ installation**. It's important
   to use the same namespace so that the two installations can see the same secrets, etc.

   <Alert severity="warning">
     <b>Make sure that you set the various `create` flags when running Helm.</b> This prevents
     $AESproductName$ $version$ from trying to configure filters that will adversely affect
     $productName$ $version$.
   </Alert>

   Typically, $productName$ $version$ was installed in the `emissary` namespace. If you installed
   $productName$ $version$ in a different namespace, change the namespace in the commands below.

   - If you do not need to set `AMBASSADOR_LABEL_SELECTOR`:

      ```bash
      helm install -n emissary \
           --set=authService.create=false \
           --set=rateLimit.create=false \
           --set=createDevPortalMappings=false \
           edge-stack datawire/edge-stack && \
      kubectl rollout status  -n emissary deployment/edge-stack -w
      ```

   - If you do need to set `AMBASSADOR_LABEL_SELECTOR`, use `--set`, for example:

      ```bash
      helm install -n emissary \
           --set=authService.create=false \
           --set=rateLimit.create=false \
           --set=createDevPortalMappings=false \
           --set emissary-ingress.env.AMBASSADOR_LABEL_SELECTOR="version-two=true" \
           edge-stack datawire/edge-stack && \
      kubectl rollout status -n emissary deployment/edge-stack -w
      ```

   <Alert severity="warning">
     You must use the <a href="https://github.com/datawire/edge-stack/"><code>edge-stack</code> Helm chart</a> to install $AESproductName$ $version$.
     Do not use the <a href="https://github.com/emissary-ingress/emissary/tree/release/v1.14/charts/ambassador"><code>ambassador</code> Helm chart</a>.
   </Alert>

3. **Test!**

   Your $AESproductName$ $version$ installation should come up running with the configuration
   resources used by $productName$ $version$, including `Listener`s and `Host`s. 

   <Alert severity="info">
     If you find that your $AESproductName$ $version$ installation and your $productName$ $version$
     installation absolutely must have resources that are only seen by one version or the
     other way, see overview section 1, "If needed, you can use labels to further isolate configurations".
   </Alert>

   **If you find that you need to roll back**, just reinstall your $productName$ $version$ CRDs
   and delete your installation of $AESproductName$ $version$.

4. **When ready, switch over to $AESproductName$ $version$.**

   You can run $productName$ $version$ and $AESproductName$ $version$ side-by-side as long as you care
   to. When you're ready to have $AESproductName$ $version$ handle traffic on its own, switch
   your original $productName$ $version$ Service to point to $AESproductName$ $version$. Use
   `kubectl edit -n emissary service emissary-ingress` and change the `selectors` to:

   ```
   app.kubernetes.io/instance: edge-stack
   app.kubernetes.io/name: edge-stack
   profile: main
   ```

   Once that is done, it's safe to remove the `emissary-ingress-admin` Service and the `emissary-ingress`
   Deployment, and to enable $AESproductName$'s filter configuration:

   ```bash
   kubectl delete -n emissary service/emissary-ingress-admin deployment/emissary-ingress
   helm upgrade -n emissary \
        --set=authService.create=true \
        --set=rateLimit.create=true \
        --set=createDevPortalMappings=true \
        edge-stack datawire/edge-stack && \
   kubectl rollout status -n emissary deployment/edge-stack -w
   ```

   You may also want to redirect DNS to the `edge-stack` Service and remove the
   `emissary-ingress` Service.

5. What's next?

   Now that you have $AESproductName$ up and running, check out the [Getting Started](../../../../../../edge-stack/latest/tutorials/getting-started) guide for recommendations on what to do next and take full advantage of its features.