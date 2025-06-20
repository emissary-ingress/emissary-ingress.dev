---
title: Service discovery and resolvers
---

Service discovery is how cloud applications and their microservices are located on the network. In a cloud environment, services are ephemeral, existing only as long as they are needed and in use, so a real-time service discovery mechanism is required. Emissary uses information from service discovery to determine where to route incoming requests.

## Emissary support for service discovery

Emissary supports different mechanisms for service discovery. These mechanisms are:

* Kubernetes service-level discovery (default).
* Kubernetes endpoint-level discovery.
* Consul endpoint-level discovery.

### Kubernetes service-level discovery

By default, Emissary uses Kubernetes DNS and service-level discovery. In a `Mapping` resource, specifying `service: foo` will prompt Emissary to look up the DNS address of the `foo` Kubernetes service. Traffic will be routed to the `foo` service. Kubernetes will then load balance that traffic between multiple pods. For more details on Kubernetes networking and how this works, see our blog post on [Session affinity, load balancing controls, gRPC-Web, and Emissary](https://blog.getambassador.io/session-affinity-load-balancing-controls-grpc-web-and-ambassador-0-52-2b916b396d0c).

### Kubernetes endpoint-level discovery

Emissary can also watch Kubernetes endpoints. This bypasses the Kubernetes service routing layer and enables the use of advanced load balancing controls such as session affinity and maglev. For more details, see the [load balancing reference](../load-balancer).

### Consul endpoint-level discovery

Emissary natively integrates with [Consul](https://www.consul.io) for endpoint-level service discovery. In this mode, Emissary obtains endpoint information from Consul. One of the primary use cases for this architecture is in hybrid cloud environments that run a mixture of Kubernetes services as well as VMs, as Consul can serve as the single global registry for all services.

## The Resolver resource

The `Resolver` resource is used to configure the discovery service strategy for Emissary.

### The Kubernetes service resolver

The Kubernetes Service Resolver configures Emissary to use Kubernetes services. If no resolver is specified, this behavior is the default. When this resolver is used, the `service.namespace` value from a `Mapping` is handed to the Kubernetes cluster's DNS resolver to determine where requests are sent.

```yaml
---
apiVersion: getambassador.io/v3alpha1
kind: KubernetesServiceResolver
metadata:
  name: kubernetes-service
```

### The Kubernetes endpoint resolver

The Kubernetes Endpoint Resolver configures Emissary to resolve Kubernetes endpoints. This enables the use of more a [advanced load balancing configuration](../load-balancer). When this resolver is used, the `Endpoint` and `EndpointSlice` resources for the `Service` defined in a `Mapping` are resolved and used to determine where requests are sent.

> **Note:** For each `Service` the resolver will first process any available `EndpointSlice` resources. If no `EndpointSlice` resources are present then the resolver will fall back to `Endpoint` resources.

```yaml
---
apiVersion: getambassador.io/v3alpha1
kind: KubernetesEndpointResolver
metadata:
  name: endpoint
```

### The Consul resolver

The Consul Resolver configures Emissary to use Consul for service discovery. When this resolver is used, the `service` defined in a `Mapping` is passed to Consul, along with the datacenter specified, to determine where requests are sent.

```yaml
---
apiVersion: getambassador.io/v3alpha1
kind: ConsulResolver
metadata:
  name: consul-dc1
spec:
  address: consul-server.default.svc.cluster.local:8500
  datacenter: dc1
```
- `address`: The fully-qualified domain name or IP address of your Consul server. This field also supports environment variable substitution.
- `datacenter`: The Consul data center where your services are registered

You may want to use an environment variable if you're running a Consul agent on each node in your cluster. In this setup, you could do the following:

```yaml
---
apiVersion: getambassador.io/v3alpha1
kind: ConsulResolver
metadata:
  name: consul-dc1
spec:
  address: "${HOST_IP}"
  datacenter: dc1
```

and then add the `HOST_IP` environment variable to your Kubernetes deployment:

```yaml
containers:
  - name: example
    image: docker.io/datawire/ambassador:$version$
    env:
      - name: HOST_IP
        valueFrom:
          fieldRef:
             fieldPath: status.hostIP
```

## Using resolvers

Once a resolver is defined, you can use them in a given `Mapping`:

```yaml
---
apiVersion: getambassador.io/v3alpha1
kind: Mapping
metadata:
  name: quote-backend
spec:
  hostname: "*"
  prefix: /backend/
  service: quote
  resolver: endpoint
  load_balancer:
    policy: round_robin
---
apiVersion: getambassador.io/v3alpha1
kind: Mapping
metadata:
  name: bar
spec:
  hostname: "*"
  prefix: /bar/
  service: https://bar:9000
  tls: client-context
  resolver: consul-dc1
  load_balancer:
    policy: round_robin
```

The YAML configuration above will configure Emissary to use Kubernetes Service Discovery to route to the Consul Service Discovery to route to the `bar` service on requests with `prefix: /bar/`.
