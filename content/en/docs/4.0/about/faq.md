---
title: Frequently Asked Questions
weight: 30
---

## General

### Why Emissary?

Kubernetes shifts application architecture for microservices, as well as the
development workflow for a full-cycle development. Emissary is designed for
the Kubernetes world with:

* Sophisticated traffic management capabilities (thanks to its use of [Envoy Proxy](https://www.envoyproxy.io)), such as load balancing, circuit breakers, rate limits, and automatic retries.
* A declarative, self-service management model built on Kubernetes Custom Resource Definitions, enabling GitOps-style continuous delivery workflows.

We've written about [the history of Emissary](https://blog.getambassador.io/building-ambassador-an-open-source-api-gateway-on-kubernetes-and-envoy-ed01ed520844), [Why Emissary In Depth](../why-ambassador), [Features and Benefits](../features-and-benefits) and about the [evolution of API Gateways](../../topics/concepts/microservices-api-gateways/).

### How is Emissary licensed?

The core Emissary is open source under the Apache Software License 2.0. The GitHub repository for the core is [https://github.com/emissary-ingress/emissary](https://github.com/emissary-ingress/emissary).

### How does Emissary use Envoy Proxy?

Emissary uses [Envoy Proxy](https://www.envoyproxy.io) as its core proxy. Envoy is an open-source, high-performance proxy originally written by Lyft. Envoy is now part of the Cloud Native Computing Foundation.

### Is Emissary production ready?

[//]: # (+FIX+ Check for OSS)

Yes. Thousands of organizations, large and small, run Emissary in production. Public users include Chick-Fil-A, ADP, Microsoft, NVidia, and AppDirect, among others.


### What's the difference between a service mesh (such as Istio) and Emissary?

Service meshes focus on routing internal traffic from service to service ("east-west"). Emissary focuses on traffic into your cluster ("north-south"). While both a service mesh and Emissary can route L7 traffic, the reality is that these use cases are quite different. Many users will integrate Emissary with a service mesh. Production customers of Emissary have integrated with Consul, Istio, and Linkerd2.

## Common Configurations

### How do I disable the default Admin mappings?

See the [Protecting the Diagnostics Interface](../../howtos/protecting-diag-access) how-to.

## Troubleshooting

### How do I get help for Emissary?

We have an online [Slack community](http://slack.cncf.io/) in the `#emissary-ingress` room. We try to help out as often as possible, although we can't promise a particular response time.

### What do I do when I get the error `no healthy upstream`?

This error means that Emissary could not connect to your backend service. Start by verifying that your backend service is actually available and responding by sending an HTTP response directly to the pod. Then verify that Emissary is routing by deploying a test service and seeing if the mapping works. Then verify that your load balancer is properly routing requests to Emissary. In general, verifying each network hop between your client and backend service is critical to finding the source of the problem.
