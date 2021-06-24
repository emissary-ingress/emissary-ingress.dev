import Alert from '@material-ui/lab/Alert';

# The `AmbassadorHost` CRD

The custom `AmbassadorHost` resource defines how $productName$ will be
visible to the outside world. It collects all the following information in a
single configuration resource:

* The hostname by which $productName$ will be reachable
* How $productName$ should handle TLS certificates
* How $productName$ should handle secure and insecure requests
* Which `AmbassadorMappings` should be associated with this `AmbassadorHost`

<Alert severity="warning">
  Remember that <code>AmbassadorListener</code> resources are&nbsp;<b>required</b>&nbsp;for a functioning 
  $productName$ installation!<br/>
  <a href="../../topics/running/ambassadorlistener">Learn more about <code>AmbassadorListener</code></a>.
</Alert>

<Alert severity="warning">
  Remember than $productName$ does not make sure that a wildcard <code>AmbassadorHost</code> exists! If the 
  wildcard behavior is needed, an <code>AmbassadorHost</code> with a <code>hostname</code> of <code>"*"</code>
  must be defined by the user.
</Alert>

A minimal `AmbassadorHost` resource, assuming no TLS configuration, would be:

```yaml
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: minimal-host
spec:
  hostname: host.example.com
```

This `AmbassadorHost` tells $productName$ to expect to be reached at `host.example.com`,
with no TLS termination, and only associating with `AmbassadorMapping`s that also set a
`hostname` that matches `host.example.com`.

Remember that an <code>AmbassadorListener</code> will also be required for this example to
be functional. Many examples of setting up `AmbassadorHost` and `AmbassadorListener` are available
in the [Configuring $productName$ to Communicate](../../../howtos/configure-communications)
document.

## Setting the `hostname`

The `hostname` element tells $productName$ which hostnames to expect. `hostname` is a DNS glob,
so all of the following are valid:

- `host.example.com`
- `*.example.com`
- `host.example.*`

The following are _not_ valid:

- `host.*.com` -- Envoy supports only prefix and suffix globs
- `*host.example.com` -- the wildcard must be its own element in the DNS name

In all cases, the `hostname` is used to match the `:authority` header for HTTP routing.
When TLS termination is active, the `hostname` is also used for SNI matching.

## Controlling Association with `AmbassadorMapping`s

An `AmbassadorMapping` will not be associated with an `AmbassadorHost` unless at least one of the following is true:

- The `AmbassadorMapping` specifies a `hostname` attribute that matches the `AmbassadorHost` in question.
- The `AmbassadorHost` specifies a `selector` that matches the `AmbassadorMapping`'s Kubernetes `label`s.

If neither of the above is true, the `AmbassadorMapping` will not be associated with the `AmbassadorHost` in 
question. This is intended to help manage memory consumption with large numbers of `AmbassadorHost`s and large
numbers of `AmbassadorMapping`s.

If the `AmbassadorHost` specifies `selector` _and_ the `AmbassadorMapping` specifies `hostname`, both must match
for the association to happen.

The `selector` is a Kubernetes [label selector](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.20/#labelselector-v1-meta), but **in 2.0, only `matchLabels` is supported**, for example:

```yaml
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: minimal-host
spec:
  hostname: host.example.com
  selector:
    matchLabels:
      examplehost: host
```

This `AmbassadorHost` will associate with the first `AmbassadorMapping` below, but not
the second:

```yaml
---
apiVersion: getambassador.io/v2
kind:  AmbassadorMapping
metadata:
  name:  use-this-mapping
  labels:
    examplehost: host
spec:
  prefix: /httpbin/
  service: http://httpbin.org
---
apiVersion: getambassador.io/v2
kind:  AmbassadorMapping
metadata:
  name:  skip-this-mapping
  labels:
    examplehost: staging
spec:
  prefix: /httpbin/
  service: http://httpbin.org
```

Future versions of $productName$ will support `matchExpressions` as well.

## Secure and insecure requests

A **secure** request arrives via HTTPS; an **insecure** request does not. By default, secure requests will be routed and insecure requests will be redirected (using an HTTP 301 response) to HTTPS. The behavior of insecure requests can be overridden using the `requestPolicy` element of an `AmbassadorHost`:

```yaml
requestPolicy:
  insecure:
    action: insecure-action
    additionalPort: insecure-port
```

The `insecure-action` can be one of:

* `Redirect` (the default): redirect to HTTPS
* `Route`: go ahead and route as normal; this will allow handling HTTP requests normally
* `Reject`: reject the request with a 400 response

```yaml
requestPolicy:
  insecure:
    additionalPort: -1   # This is how to disable the default redirection from 8080.
```

Some special cases to be aware of here:

* **Case matters in the actions:** you must use e.g. `Reject`, not `reject`.
* The `X-Forwarded-Proto` header is honored when determining whether a request is secure or insecure. For more information, see "Load Balancers, the `AmbassadorHost` Resource, and `X-Forwarded-Proto`" below.
* ACME challenges with prefix `/.well-known/acme-challenge/` are always forced to be considered insecure, since they are not supposed to arrive over HTTPS.
* $AESproductName$ provides native handling of ACME challenges. If you are using this support, $AESproductName$ will automatically arrange for insecure ACME challenges to be handled correctly. If you are handling ACME yourself - as you must when running $OSSproductName$ - you will need to supply appropriate `AmbassadorHost` resources and Mappings to correctly direct ACME challenges to your ACME challenge handler.

## TLS settings

The `AmbassadorHost` is responsible for high-level TLS configuration in $productName$. There are
several settings covering TLS:

### `tlsSecret` enables TLS termination

`tlsSecret` specifies a Kubernetes `Secret` is **required** for any TLS termination to occur. No matter what other TLS
configuration is present, TLS termination will not occur if `tlsSecret` is not specified.

The following `AmbassadorHost` will configure $productName$ to read a `Secret` named 
`tls-cert` for a certificate to use when terminating TLS.

```yaml
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: example-host
spec:
  hostname: host.example.com
  acmeProvider:
    authority: none
  tlsSecret:
    name: tls-cert
```

### `tlsContext` links to a `TLSContext` for additional configuration

`tlsContext` specifies a [`TLSContext`](#) to use for additional TLS information. Note that you **must** still 
define `tlsSecret` for TLS termination to happen. It is an error to supply both `tlsContext` and `tls`.

See the [TLS discussion](../tls) for more details.

### `tls` allows manually providing additional configuration

`tls` allows specifying most of the things a `TLSContext` can, inline in the `AmbassadorHost`. Note that you **must** still 
define `tlsSecret` for TLS termination to happen. It is an error to supply both `tlsContext` and `tls`.

See the [TLS discussion](../tls) for more details.

## Load balancers, the `AmbassadorHost` resource, and `X-Forwarded-Proto`

In a typical installation, $productName$ runs behind a load balancer. The
configuration of the load balancer can affect how $productName$ sees requests
arriving from the outside world, which can in turn can affect whether $productName$
considers the request secure or insecure. As such:

- **We recommend layer 4 load balancers** unless your workload includes
  long-lived connections with multiple requests arriving over the same
  connection. For example, a workload with many requests carried over a small
  number of long-lived gRPC connections.
- **$productName$ fully supports TLS termination at the load balancer** with a single exception, listed below.
- If you are using a layer 7 load balancer, **it is critical that the system be configured correctly**:
  - The load balancer must correctly handle `X-Forwarded-For` and `X-Forwarded-Proto`.
  - The `xff_num_trusted_hops` element in the `ambassador` module must be set to the number of layer 7 load balancers the request passes through to reach $productName$ (in the typical case, where the client speaks to the load balancer, which then speaks to $productName$, you would set `xff_num_trusted_hops` to 1). If `xff_num_trusted_hops` remains at its default of 0, the system might route correctly, but upstream services will see the load balancer's IP address instead of the actual client's IP address.

It's important to realize that Envoy manages the `X-Forwarded-Proto` header such that it **always** reflects the most trustworthy information Envoy has about whether the request arrived encrypted or unencrypted. If no `X-Forwarded-Proto` is received from downstream, **or if it is considered untrustworthy**, Envoy will supply an `X-Forwarded-Proto` that reflects the protocol used for the connection to Envoy itself. The `xff_num_trusted_hops` element, although its name reflects `X-Forwarded-For`, is also used when determining trust for `X-Forwarded-For`, and it is therefore important to set it correctly. Its default of 0 should always be correct when $productName$ is behind only layer 4 load balancers; it should need to be changed **only** when layer 7 load balancers are involved.

### CRD specification

The `AmbassadorHost` CRD is formally described by its protobuf specification. Developers who need access to the specification can find it [here](https://github.com/emissary-ingress/emissary/blob/master/api/getambassador.io/v2/Host.proto).
