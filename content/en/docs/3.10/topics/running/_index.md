---
title: "Running"
description: "This section of the documentation provides instructions on running Emissary in a production environment"
---

# Running Emissary in production

This section of the documentation is designed for operators and site reliability engineers who are managing the deployment of Emissary. Learn more below:

* *Global Configuration:* The [Ambassador module](ambassador) is used to set system-wide configuration.
* *Exposing Emissary to the Internet:* The [`Listener` CRD](listener) defines which ports are exposed, including their protocols and security models. The [`Host` CRD](host-crd) defines how Emissary manages TLS, domains, and such.
* *Load Balancing:* Emissary supports a number of different [load balancing strategies](load-balancer) as well as different ways to configure [service discovery](resolvers)
* [Gzip Compression](gzip)
* *Deploying Emissary:* On [Amazon Web Services](ambassador-with-aws) | [Google Cloud](ambassador-with-gke) | [general security and operational notes](running), including running multiple emissaries on a cluster
* *TLS/SSL:* [Simultaneously Routing HTTP and HTTPS](tls/cleartext-redirection#cleartext-routing) | [HTTP -> HTTPS Redirection](tls/cleartext-redirection#http-https-redirection) | [Mutual TLS](tls/mtls) | [TLS origination](tls/origination)
* *Statistics and Monitoring:* [Integrating with Prometheus, DataDog, and other monitoring systems](statistics)
* *Extending Emissary* Emissary can be extended with custom plug-ins that connect via HTTP/gRPC interfaces. [Custom Authentication](services/auth-service) | [The External Auth protocol](services/ext-authz) | [Custom Logging](services/log-service) | [Rate Limiting](services/rate-limit-service) | [Distributed Tracing](services/tracing-service)
* *Troubleshooting:* [Diagnostics](diagnostics) | [Debugging](debugging)
* *Scaling Emissary:* [Scaling Emissary](scaling)
* *Ingress:* Emissary can function as an [Ingress Controller](ingress-controller)
* *Error Response Overrides:* Emissary can override 4xx and 5xx responses with [custom response bodies](custom-error-responses)
