---
title: "Quick Start"
weight: 5
---

**We recommend using Helm** to install Emissary. If you really need to
work directly with YAML, use `helm template` to generate the YAML
manifests you need.

## Installing Emissary

Installing Emissary is a two-step process: first, you install the Custom
Resource Definitions (CRDs) that Emissary uses to configure itself, and
then you install Emissary itself. There are two different paths here,
depending on whether you're installing Emissary for the first time, or
upgrading from an earlier version where you're using older versions of
the CRDs.

### Path 1: Fresh Install

**If you have never run Emissary before, this path is for you!**

(This is also the path for you if you're running Emissary 3 and you're
absolutely, positively, 100% _certain_ that you have no v1 or v2 Emissary
CRDs in your cluster. If you're not _completely_ certain, though, check
out the upgrade path below.)

Starting from scratch with Emissary is straightforward: since there won't
be any concerns about old CRD versions, the default installation is
exactly what you want. First, install the CRDs:

```bash
helm install emissary-crds \
 oci://ghcr.io/emissary-ingress/emissary-crds-chart --version=4.0.0-rc.1 \
 --wait
```

This will install only the most recent (v3alpha1) CRD versions, without
any conversion-webhook stuff.

Next up, install Emissary itself:

```bash
helm install emissary \
 --namespace emissary --create-namespace \
 oci://ghcr.io/emissary-ingress/emissary-ingress --version=4.0.0-rc.1 \
 --wait
```

This will take a minute or so to finish, since it'll wait for Emissary to
be fully running, but once it's done, you're ready to go!

### Path 2: Upgrading from an earlier Emissary

This time, we're going to deliberately install the old CRD versions and
the conversion webhook:

```bash
helm install emissary-crds \
 --namespace emissary-system --create-namespace \
 oci://ghcr.io/emissary-ingress/emissary-crds-chart --version=4.0.0-rc.1 \
 --set enableLegacyVersions=true \
 --set enableV1=true \
 --wait
```

(If you're sure you have no v1 CRDs, you can skip the `--set enableV1=true` line.)

This will install all the versions of the CRDs (v1, v2, and v3alpha1) and
also install the conversion webhook into the `emissary-system` namespace.
Once that's done, you'll install Emissary itself, but you'll need to tell
it to be sure the conversion webhook is running before fully starting up:

```bash
helm install emissary \
 --namespace emissary --create-namespace \
 oci://ghcr.io/emissary-ingress/emissary-ingress --version=4.0.0-rc.1 \
 --set waitForApiext.enabled=true \
 --wait
```

This will take a minute or so to finish, since it'll wait for Emissary to
be fully running, but once it's done, you're ready to go!

### Listing Emissary with Helm

The `emissary-crds-chart` and `emissary-ingress` installations will
always end up in different namespaces, so `helm list -A` is the easiest
way to see everything related to Emissary.

## Using Emissary

After installing Emissary, you'll have a running Emissary behind the
Service named `emissary` in the `emissary` namespace. How exactly you
connect to that Service will vary with your cluster provider, but you can
start with the following:

```bash
kubectl get svc -n emissary emissary
```

That should get you started. Or, of course, you can use something like this:

```bash
kubectl port-forward -n emissary svc/emissary 8080:80
```

(after you configure a Listener!) and then talk to localhost:8080 with any
kind of cluster.

# Using Faces for a sanity check

[Faces Demo]: https://github.com/buoyantio/faces-demo

If you like, you can continue by using the [Faces Demo] as a quick sanity
check. First, install Faces itself using Helm:

```bash
helm install faces \
 --namespace faces --create-namespace \
 oci://ghcr.io/buoyantio/faces-chart --version 2.0.0 \
 --wait
```

Next, you'll need to configure Emissary to route to Faces. First, we'll do the
basic configuration to tell Emissary to listen for HTTP traffic:

```bash
kubectl apply -f - <<EOF
---
apiVersion: getambassador.io/v3alpha1
kind: Listener
metadata:
  name: ambassador-https-listener
spec:
  port: 8443
  protocol: HTTPS
  securityModel: XFP
  hostBinding:
    namespace:
      from: ALL
---
apiVersion: getambassador.io/v3alpha1
kind: Listener
metadata:
  name: ambassador-http-listener
spec:
  port: 8080
  protocol: HTTP
  securityModel: XFP
  hostBinding:
    namespace:
      from: ALL
---
apiVersion: getambassador.io/v3alpha1
kind: Host
metadata:
  name: wildcard-host
spec:
  hostname: "*"
  requestPolicy:
    insecure:
      action: Route
EOF
```

(This actually supports both HTTPS and HTTP, but since we haven't set up TLS
certificates, we'll just stick with HTTP.)

Next, we need two Mappings:

| Prefix    | Routes to Service | in Namespace |
| --------- | ----------------- | ------------ |
| `/faces/` | `faces-gui`       | `faces`      |
| `/face/`  | `face`            | `faces`      |

```bash
kubectl apply -f - <<EOF
---
apiVersion: getambassador.io/v3alpha1
kind: Mapping
metadata:
  name: gui-mapping
  namespace: faces
spec:
  hostname: "*"
  prefix: /faces/
  service: faces-gui.faces
  rewrite: /
  timeout_ms: 0
---
apiVersion: getambassador.io/v3alpha1
kind: Mapping
metadata:
  name: face-mapping
  namespace: faces
spec:
  hostname: "*"
  prefix: /face/
  service: face.faces
  timeout_ms: 0
EOF
```

Once that's done, then you'll be able to access the Faces Demo at `/faces/`,
on whatever IP address or hostname your cluster provides for the `emissary`
Service. Or you can port-forward as above and access it at
`http://localhost:8080/faces/`.

# Next Steps

Further explore some of the concepts you learned about in this article:
* [`Mapping` resource](../topics/using/intro-mappings/): routes traffic from the edge of your cluster to a Kubernetes service
* [`Host` resource](../topics/running/host-crd/): sets the hostname by which Emissary will be accessed and secured with TLS certificates

Learn more about [how developers use Emissary](../topics/using/) to manage
edge policies.

Learn more about [how site reliability engineers and operators run Emissary](../topics/running/)
in production environments.

To learn how Emissary works, use cases, best practices, and more, check out the [Emissary Story](../about/why-ambassador).
