---
    description: In this guide, we'll walk through the process of deploying $productName$ in Kubernetes for ingress routing.
---

import Alert from '@material-ui/lab/Alert';

# Install manually

<Alert severity="info">
  We're pleased to introduce $productName$ 2.0 as a <b>developer preview</b>; our latest
  general-availability release is <a href="../../../1.13">1.13</a>.<br/>
  <br/>
  The 2.X family introduces a number of changes to allow $productName$ to more gracefully
  handle larger installations (including multitenant or multiorganizational installations),
  reduce memory footprint, and improve performance. However, <b>some configuration has
  changed</b> between 1.X and 2.X: if you're currently running 1.X, <b>please</b> read the&nbsp;
  <a href="migrate-to-version-2">migration guide</a> before trying to install 2.0.
</Alert>

In this guide, we'll walk you through installing $productName$ in your Kubernetes cluster.

The manual install process does not allow for as much control over configuration
as the [Helm install method](../helm), so if you need more control over your $productName$
installation, it is recommended that you use helm.

## Before you begin

$productName$ is designed to run in Kubernetes for production. The most essential requirements are:

* Kubernetes 1.11 or later
* The `kubectl` command-line tool

## Install or Upgrade with YAML

$productName$ is typically deployed to Kubernetes from the command line. If you don't have Kubernetes, you should use our [Docker](../docker) image to deploy $productName$ locally.

1. In your terminal, run the following command:

    ```
    kubectl create namespace $productNamespace$ || true
    kubectl apply -f https://app.getambassador.io/yaml/emissary/$version$/emissary-crds.yaml && \
    kubectl apply -f https://app.getambassador.io/yaml/emissary/$version$/emissary-ingress.yaml && \
    kubectl -n $productNamespace$ wait --for condition=available --timeout=90s deploy $productDeploymentName$
    ```

2. Determine the IP address or hostname of your cluster by running the following command:

    ```
    kubectl get -n $productNamespace$ service $productDeploymentName$ -o "go-template={{range .status.loadBalancer.ingress}}{{or .ip .hostname}}{{end}}"
    ```

    Your load balancer may take several minutes to provision your IP address. Repeat the provided command until you get an IP address.
