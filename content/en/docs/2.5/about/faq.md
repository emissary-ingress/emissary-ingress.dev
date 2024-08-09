---
title: Frequently Asked Questions
---

## General

### Why Emissary?

Kubernetes shifts application architecture for microservices, as well as the
development workflow for a full-cycle development. Emissary is designed for
the Kubernetes world with:

* Sophisticated traffic management capabilities (thanks to its use of [Envoy Proxy](https://www.envoyproxy.io)), such as load balancing, circuit breakers, rate limits, and automatic retries.
* A declarative, self-service management model built on Kubernetes Custom Resource Definitions, enabling GitOps-style continuous delivery workflows.

We've written about [the history of Emissary](https://blog.getambassador.io/building-ambassador-an-open-source-api-gateway-on-kubernetes-and-envoy-ed01ed520844), [Why Emissary In Depth](../why-ambassador), [Features and Benefits](../features-and-benefits) and about the [evolution of API Gateways](../../topics/concepts/microservices-api-gateways/).

### What's the difference between Emissary and Ambassador Edge Stack?

Emissary is a CNCF Incubating project and provides the open-source core of Ambassador Edge Stack. Originally we called Emissary the "Ambassador API Gateway", but as the project evolved, we realized that the functionality we were building had extended far beyond an API Gateway. In particular, the Ambassador Edge Stack is intended to provide all the functionality you need at the edge -- hence, an "edge stack." This includes an API Gateway, ingress controller, load balancer, developer portal, and more.

### How is Ambassador Edge Stack licensed?

The core Emissary is open source under the Apache Software License 2.0. The GitHub repository for the core is [https://github.com/emissary-ingress/emissary](https://github.com/emissary-ingress/emissary). Some additional features of the Ambassador Edge Stack (e.g., Single Sign-On) are not open source and available under a proprietary license.

### Can I use the add-on features for Ambassador Edge Stack for free?

Yes! The core functionality of the Ambassador Edge Stack is free and has no limits whatsoever. If you wish to use one of our additional, proprietary features such as Single Sign-On, you can get a free community license for up to 5 requests per second by registering your connected Ambassador Edge Stack installation in [Ambassador Cloud](https://app.getambassador.io/cloud/). Please contact [sales](/contact-us/) if you need more than 5 RPS.

For more details on core unlimited features and premium features, see the [editions page](/editions).

### How does Emissary use Envoy Proxy?

Emissary uses [Envoy Proxy](https://www.envoyproxy.io) as its core proxy. Envoy is an open-source, high-performance proxy originally written by Lyft. Envoy is now part of the Cloud Native Computing Foundation.

### Is Emissary production ready?

[//]: # (+FIX+ Check for OSS)

Yes. Thousands of organizations, large and small, run Emissary in production.
Public users include Chick-Fil-A, ADP, Microsoft, NVidia, and AppDirect, among others.

### What is the performance of Emissary?

There are many dimensions to performance. We published a benchmark of [Emissary performance on Kubernetes](/resources/envoyproxy-performance-on-k8s/). Our internal performance regressions cover many other scenarios; we expect to publish more data in the future.

### What's the difference between a service mesh (such as Istio) and Emissary?

[//]: # (+FIX+ Check for OSS)

Service meshes focus on routing internal traffic from service to service
("east-west"). Emissary focuses on traffic into your cluster ("north-south").
While both a service mesh and Emissary can route L7 traffic, the reality is that
these use cases are quite different. Many users will integrate Emissary with a
service mesh. Production customers of Emissary have integrated with Consul,
Istio, and Linkerd2.

## Common Configurations

### How do I disable the default Admin mappings?

See the [Protecting the Diagnostics Interface](../../howtos/protecting-diag-access) how-to.

## Troubleshooting

### How do I get help for Emissary?

We have an online [Slack community](http://a8r.io/slack) with thousands of
users. We try to help out as often as possible, although we can't promise a
particular response time. If you need a guaranteed SLA, we also have commercial
contracts. [Contact sales](/contact-us/) for more information.

### What do I do when I get the error `no healthy upstream`?

This error means that Emissary could not connect to your backend service.
Start by verifying that your backend service is actually available and
responding by sending an HTTP response directly to the pod. Then, verify that
Emissary is routing by deploying a test service and seeing if the mapping
works. Then, verify that your load balancer is properly routing requests to
Emissary. In general, verifying each network hop between your client and
backend service is critical to finding the source of the problem.
