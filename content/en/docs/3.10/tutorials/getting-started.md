---
title: "Getting Started with Emissary"
description: "Learn how to install Emissary with either Helm or kubectl to get started routing traffic from the edge of your Kubernetes cluster to your services..."
---

# Emissary quick start

<div class="docs-article-toc">
<h3>Contents</h3>

- [1. Installation](#1-installation)
- [2. Routing traffic from the edge](#2-routing-traffic-from-the-edge)
- [What's next?](#img-classos-logo-srcimageslogopng-whats-next)

</div>

## 1. Installation

We'll start by installing Emissary into your cluster.

**We recommend using Helm** but there are other options below to choose from.

<GettingStartedEmissary21Tabs version="$version$" />

## 2. Routing traffic from the edge

Emissary uses Kubernetes Custom Resource Definitions (CRDs) to declaratively define its desired state. The workflow you are going to build uses a simple demo app, a **`Listener` CRD**, and a **`Mapping` CRD**. The `Listener` CRD tells Emissary what port to listen on, and the `Mapping` CRD tells Emissary how to route incoming requests by host and URL path from the edge of your cluster to Kubernetes services.

1. Start by creating a `Listener` resource for HTTP on port 8080:

   ```
   kubectl apply -f - <<EOF
   ---
   apiVersion: getambassador.io/v3alpha1
   kind: Listener
   metadata:
     name: $productDeploymentName$-listener-8080
     namespace: $productNamespace$
   spec:
     port: 8080
     protocol: HTTP
     securityModel: XFP
     hostBinding:
       namespace:
         from: ALL
   EOF
   ```

     <Alert severity="info">
       This <code>Listener</code> will associate with <i>all</i> <code>Host</code>s in your cluster. This is fine for the quickstart, but is likely not what you really want for production use.<br/>
       <br/>
       <a href="../../topics/running/listener">Learn more about <code>Listener</code></a>.<br/>
       <a href="../../topics/running/host-crd">Learn more about <code>Host</code></a>.
     </Alert>

2. Apply the YAML for the "Quote" service.

  ```
  kubectl apply -f https://app.getambassador.io/yaml/v2-docs/$ossVersion$/quickstart/qotm.yaml
  ```

  <Alert severity="info">The Service and Deployment are created in your default namespace. You can use <code>kubectl get services,deployments quote</code> to see their status.</Alert>

3. Generates the YAML for a `Mapping` to tell Emissary to route all traffic inbound to the `/backend/` path to the `quote` Service.
    - **Path Matching**: `/backend/`
    - **OpenAPI Docs**: `/.ambassador-internal/openapi-docs`

  **Apply this YAML to your target cluster now.**

  ```yaml
  kubectl apply -f - <<EOF
  ---
  apiVersion: getambassador.io/v3alpha1
  kind: Mapping
  metadata:
    name: quote-backend
  spec:
    hostname: "*"
    prefix: /backend/
    service: quote
    docs:
      path: "/.ambassador-internal/openapi-docs"
  EOF
  ```

4. Store the Emissary load balancer IP address to a local environment variable. You will use this variable to test access to your service.

  ```
  export LB_ENDPOINT=$(kubectl -n $productNamespace$ get svc  $productDeploymentName$ \
    -o "go-template={{range .status.loadBalancer.ingress}}{{or .ip .hostname}}{{end}}")
  ```

5. Test the configuration by accessing the service through the Emissary load balancer:

  ```
  $ curl -i http://$LB_ENDPOINT/backend/

    HTTP/1.1 200 OK
    content-type: application/json
    date: Wed, 23 Jun 2021 15:49:02 GMT
    content-length: 137
    x-envoy-upstream-service-time: 0
    server: envoy

    {
        "server": "ginormous-kumquat-7mkgucxo",
        "quote": "Abstraction is ever present.",
        "time": "2021-06-23T15:49:02.255042819Z"
    }
  ```

<Alert severity="success"><b>Victory!</b> You have created your first Emissary <code>Listener</code> and <code>Mapping</code>, routing a request from your cluster's edge to a service!</Alert>

## <img class="os-logo" src="/images/logo.png"/> What's next?

Explore some of the popular tutorials on Emissary:

* [Configuring Emissary communications](../../howtos/configure-communications): configure how Emissary handles communication with clients
* [Intro to `Mappings`](../../topics/using/intro-mappings/): declaratively routes traffic from
the edge of your cluster to a Kubernetes service
* [`Listener` resource](../../topics/running/listener/): configure ports, protocols, and security options for your ingress.
* [`Host` resource](../../topics/running/host-crd/): configure a hostname and TLS options for your ingress.

Emissary has a comprehensive range of [features](/features/) to
support the requirements of any edge microservice.

To learn more about how Emissary works, read the [Emissary Story](../../about/why-ambassador).
