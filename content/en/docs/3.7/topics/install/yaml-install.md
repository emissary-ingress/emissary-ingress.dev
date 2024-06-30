---
title: YAML
description: In this guide, we'll walk through the process of deploying Emissary in Kubernetes for ingress routing.
---

# Install manually

<Alert severity="warning">

  To migrate from Emissary 1.X to Emissary 2.X, see the
  [Emissary migration matrix](../migration-matrix/).  This guide
  **will not work** for that, due to changes to the configuration
  resources used for Emissary 2.X.

</Alert>

In this guide, we'll walk you through installing Emissary in your Kubernetes cluster.

The manual install process does not allow for as much control over configuration
as the [Helm install method](../helm), so if you need more control over your Emissary
installation, it is recommended that you use helm.

## Before you begin

Emissary is designed to run in Kubernetes for production. The most essential requirements are:

* Kubernetes 1.11 or later
* The `kubectl` command-line tool

## Install with YAML

Emissary is typically deployed to Kubernetes from the command line. If you don't have Kubernetes, you should use our [Docker](../docker) image to deploy Emissary locally.

1. In your terminal, run the following command:

    ```
    kubectl create namespace $productNamespace$ || true
    kubectl apply -f https://app.getambassador.io/yaml/emissary/$version$/emissary-crds.yaml && \
    kubectl apply -f https://app.getambassador.io/yaml/emissary/$version$/emissary-emissaryns.yaml && \
    kubectl -n $productNamespace$ wait --for condition=available --timeout=90s deploy $productDeploymentName$
    ```

   <Alert severity="info">
     Emissary $version$ includes a Deployment in the $productNamespace$ namespace
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
    There is a known issue with the <code>emissary-apiext</code> service that impacts all Emissary 2.x and 3.x users. Specifically, the TLS certificate used by apiext expires one year after creation and does not auto-renew. All users who are running Emissary/Emissary 2.x or 3.x with the apiext service should proactively renew their certificate as soon as practical by running <code>kubectl delete --all secrets --namespace=emissary-system</code> to delete the existing certificate, and then restart the <code>emissary-apiext</code> deployment with <code>kubectl rollout restart deploy/emissary-apiext -n emissary-system</code>.
    This will create a new certificate with a one year expiration. We will issue a software patch to address this issue well before the one year expiration. Note that certificate renewal will not cause any downtime.
   </Alert>

2. Determine the IP address or hostname of your cluster by running the following command:

    ```
    kubectl get -n $productNamespace$ service $productDeploymentName$ -o "go-template={{range .status.loadBalancer.ingress}}{{or .ip .hostname}}{{end}}"
    ```

    Your load balancer may take several minutes to provision your IP address. Repeat the provided command until you get an IP address.

3. Next Steps

   Emissary shold now be successfully installed and running, but in order to get started deploying Services and test routing to them you need to configure a few more resources.

   - [The `Listener` Resource](../../running/listener/) is required to configure which ports the Emissary pods listen on so that they can begin responding to requests.
   - [The `Mapping` Resouce](../../using/intro-mappings/) is used to configure routing requests to services in your cluster.
   - [The `Host` Resource](../../running/host-crd/) configures TLS termination for enablin HTTPS communication.
   - Explore how Emissary [configures communication with clients](../../../howtos/configure-communications)

  <Alert severity="info">
     We strongly recommend following along with our <a href="../../../tutorials/getting-started">Quickstart Guide</a> to get started by creating a <code>Listener</code>, deploying a simple service to test with, and setting up a <code>Mapping</code> to route requests from Emissary to the demo service.
  </Alert>

## Upgrading an existing installation

See the [migration matrix](../migration-matrix) for instructions about upgrading
Emissary.

<img referrerpolicy="no-referrer-when-downgrade" src="https://static.scarf.sh/a.png?x-pxid=a6dee6ae-7a66-47cb-ab70-2e576b24e521" alt="" style="display: none;"/>
