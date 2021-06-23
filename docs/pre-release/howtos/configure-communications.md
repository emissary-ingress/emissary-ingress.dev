Configuring $productName$ to Communicate
========================================

For $productName$ to do its job of managing network communications for your services, it first needs to know how its own communications should be set up. This is handled by a combination of resources: the `AmbassadorListener`, the `AmbassadorHost`, and the `TLSContext`.

- `AmbassadorListener`s define where, and how, $productName$ should listen for requests from the network.
- `AmbassadorHost`s can be associated with one or more `AmbassadorListener`s. They define which hostnames $productName$ should care about, and how to handle different kinds of requests for those hosts.
- `TLSContext`s can be associated with one or more `AmbassadorHost`s. They define whether, and how, $productName$ will manage TLS certificates and options.

Once the basic communications setup is in place, $productName$ `AmbassadorMapping`s and `AmbassadorTCPMapping`s can be associated with `AmbassadorHost`s to actually do routing.

References 
----------

It is a very good idea to be familiar with the CRDs in play here:

- [`AmbassadorListener`](../../topics/running/ambassadorlistener)
- [`AmbassadorHost`](../../topics/running/host-crd)
- [`TLSContext`](../../topics/running/tls/#tlscontext)

Examples / Cookbook
-------------------

### Unconfigured Defaults With TLS Active

If $productName$ has been configured such that it's possible to terminate TLS, but no `Listeners` are present, two default `AmbassadorListener`s are created:

```yaml
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorListener
metadata:
  name: default-http
spec:
  port: 8080
  securityModel: XFP
  protocol: HTTPS # NOT A TYPO, see below
  l7Depth: 0
  hostBinding:
    namespace:
      from: SELF 
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorListener
metadata:
  name: default-https
spec:
  port: 8443
  securityModel: XFP
  protocol: HTTPS
  l7Depth: 0
  hostBinding:
    namespace:
      from: SELF
```

If no `AmbassadorHost`s have been defined, but a fallback secret is present, we create a default `AmbassadorHost` using the fallback secret:

```yaml
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: default-host
spec:
  hostname: "*"
  acmeProvider:
    authority: none
  tlsSecret: 
    name: fallback-secret
  requestPolicy:
    insecure:
      action: Redirect
```

(If no `AmbassadorHost`s have been defined and no fallback secret is present, look below to "Unconfigured Defaults With No TLS" for the default `AmbassadorHost` setting.)

- The effect of the default `AmbassadorListener`s is to listen for either HTTP or HTTPS on ports 8080 and 8443.
- The effect of the default `AmbassadorHost` is to route HTTPS and redirect HTTP to HTTPS.
- Overall, the defaults boil down to TLS by default, relying on the fallback self-signed certificate if no other certificates are provided.
- **Note well** that if you manually define termination `TLSContext`s, but you don't define an `AmbassadorHost`, your `TLSContext`s will be ignored. We will log a warning in this situation.
   - Also note that if, for some reason, a `TLSContext` using the fallback cert is defined with no defined `AmbassadorHost`s, then the `TLSContext` will be ignored, even though the fallback cert is in use. This is not expected to be common.

### Unconfigured Defaults With No TLS

If the system cannot terminate TLS, the defaults are a bit simpler:

```yaml
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorListener
metadata:
  name: default-http
spec:
  port: 8080
  securityModel: INSECURE
  protocol: HTTP
  l7Depth: 0
  hostBinding:
    namespace:
      from: SELF
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: default-host
spec:
  hostname: "*"
  acmeProvider:
    authority: none
  requestPolicy:
    insecure:
      action: Route
```

- This will result in a cleartext-only $productName$ configuration, with only port 8080 active.
   - See the "Cleartext Only" section below for why this is important.

### TLS using ACME ($AESproductName$ only)

This scenario uses ACME to get certificates for `foo.example.com` and `bar.example.com`. HTTPS traffic to either host is routed; HTTP traffic to `foo.example.com` will be redirected to HTTPS, but HTTP traffic to `bar.example.com` will be rejected outright.

Since this example uses ACME, it is only supported in $AESproductName$.

```yaml
---
apiversion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: foo-host
spec:
  hostname: foo.example.com
  acmeProvider:
    email: julian@example.com
---
apiversion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: bar-host
spec:
  hostname: bar.example.com
  acmeProvider:
    email: julian@example.com
  requestPolicy:
    insecure:
      action: Reject
```

(`AmbassadorMapping`s are not shown.)

- Since no `AmbassadorListener`s are defined, $AESproductName$ will create default `AmbassadorListener`s on ports 8080 and 8443.
   - Both will accept HTTPS and HTTP, and the protocol will dictate whether the requests are secure (HTTPS) or insecure (HTTP).
- `foo-host` defaults to ACME with Let's Encrypt, since `acmeProvider.authority` is not provided.
- `foo-host` defaults to redirecting insecure requests, since the default for `requestPolicy.insecure.action` is `Redirect`.
- `bar-host` uses Let's Encrypt as well, but it will reject insecure requests.

### Bring Your Own Certificate

This scenario uses TLS, but no ACME: instead, the certificate is in a Kubernetes `Secret`. HTTPS traffic to either `foo.example.com` or `bar.example.com` is routed, but this time `foo.example.com` will redirect HTTP requests, while `bar.example.com` will route them.

Since this example does not use ACME, it is supported in $productName$ as well as $AESproductName$.

```yaml
---
apiversion: v1
kind: Secret
type: kubernetes.io/tls
metadata:
  name: wildcard-example-secret
data:
  tls.crt: -wildcard here-
  tls.key: -wildcard here-
---
apiversion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: foo-host
spec:
  hostname: foo.example.com
  tlsSecret:
    name: wildcard-example-secret
---
apiversion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: bar-host
spec:
  hostname: bar.example.com
  tlsSecret:
    name: wildcard-example-secret
  requestPolicy:
    insecure:
      action: Route
```

- The default `AmbassadorListener`s are still created.
- `foo-host` and `bar-host` simply reference the `tlsSecret` to use for termination.
   - `wildcard-example-secret` is assumed to have a wildcard `CN`, or to include both `foo.example.com` and `bar.example.com` in its `SAN` list.
   - `foo-host` and `bar-host` could also reference different secrets, of course.

### Bring Your Own TLSContext

If you need to share other TLS settings between two `AmbassadorHost`s, you can reference a `TLSContext` as well as the `tlsSecret`. This example is the same as "Bring Your Own Certificate", but we use a `TLSContext` to set `ALPN` information.

```yaml
---
apiversion: v1
kind: Secret
type: kubernetes.io/tls
metadata:
  name: wildcard-example-secret
data:
  tls.crt: -wildcard here-
  tls.key: -wildcard here-
---
apiversion: getambassador.io/v2
kind: TLSContext
metadata:
  name: example-context
spec:
  secret: wildcard-example-secret
  alpn_protocols: [h2, istio]
---
apiversion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: foo-host
spec:
  hostname: foo.example.com
  tlsContext:
    name: example-context
  tlsSecret:
    name: wildcard-example-secret
---
apiversion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: bar-host
spec:
  hostname: bar.example.com
  tlsContext:
    name: example-context
  tlsSecret:
    name: wildcard-example-secret
  requestPolicy:
    insecure:
      action: Route
```

### ACME With a TLSContext ($AESproductName$ Only)

In $AESproductName$, you can use a `TLSContext` with ACME as well. This example is the same as "TLS using ACME", but we use a `TLSContext` to set `ALPN` information. Again, ACME is only supported in $AESproductName$.

```yaml
---
apiversion: getambassador.io/v2
kind: TLSContext
metadata:
  name: example-context
spec:
  secret: example-acme-secret
  alpn_protocols: [h2, istio]
---
apiversion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: foo-host
spec:
  hostname: foo.example.com
  acmeProvider:
    email: julian@example.com
  tlsContext:
    name: example-context
  tlsSecret:
    name: example-acme-secret
```

- Note that we don't provide the `Secret`: the ACME client will create it for us.

### Cleartext Only

This scenario doesn't do TLS at all, but will accept cleartext HTTP to any `example.com` domain.

```yaml
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorListener
metadata:
  name: cleartext-listener
spec:
  protocol: HTTP
  port: 8080
  hostBinding:     # This may well need editing for your case!
    namespace:
      from: SELF
---
apiversion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: wildcard-host
spec:
  hostname: "*.example.com"
  requestPolicy:
    insecure:
      action: Route

```

- Since we provide an `AmbassadorListener`, no default `AmbassadorListener`s are created.
   - This is important in this scenario! If the default `AmbassadorListener`s are created, they are created to _allow_ TLS.
      - In $AESproductName$, this will result in TLS being accepted using the fallback certificate.
      - In $productName$, since there is no fallback certficate, it will result in strange TLS errors.
- Using an insecure action of `Route` is necessary! otherwise requests will be redirected and nothing will catch them.

### Using an L7 Load Balancer to Terminate TLS

In this scenario, a layer 7 load balancer ahead of $productName$ will terminate TLS, so $productName$ will always see HTTP with a known good `X-Forwarded-Protocol`. We'll use that to route HTTPS and redirect HTTP.

```yaml
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorListener
metadata:
  name: lb-listener
spec:
  protocol: HTTP
  port: 8443
  l7Depth: 1
  hostBinding:     # This may well need editing for your case!
    namespace:
      from: SELF
```

- We set `l7Depth` to 1 to indicate that there's a single trusted L7 load balancer ahead of us.
- We specifically set this AmbassadorListener to HTTP-only, but we stick with port 8443 just because we expect people setting up TLS at all to expect to use port 8443. (There's nothing special about the port number, pick whatever you like.)
- This is a rare case where we don't need an `AmbassadorHost`: the default `AmbassadorHost` accepts traffic for any hostname, routes HTTPS, and redirects HTTP.

### Using a Split L4 Load Balancer to Terminate TLS

Here, we assume that $productName$ is behind a load balancer setup that handles TLS at layer 4:

- Incoming cleartext traffic is forwarded to $productName$ on port 8080.
- Incoming TLS traffic is terminated at the load balancer, then forwarded to $productName$ _as cleartext_ on port 8443.
- This might involve multiple L4 load balancers, but the actual number doesn't matter.
- The actual port numbers we use don't matter either, as long as $productName$ and the load balancer(s) agree on which port is for which traffic.

We're going to route HTTPS for both `foo.example.com` and `bar.example.com`, redirect HTTP for `foo.example.com`, and reject HTTP for `bar.example.com`.

```yaml
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorListener
metadata:
  name: split-lb-one-listener
spec:
  protocol: HTTP
  port: 8080
  securityModel: INSECURE
  hostBinding:     # This may well need editing for your case!
    namespace:
      from: SELF
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorListener
metadata:
  name: split-lb-two-listener
spec:
  protocol: HTTP
  port: 8443
  securityModel: SECURE
  hostBinding:     # This may well need editing for your case!
    namespace:
      from: SELF
---
apiversion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: foo-host
spec:
  hostname: foo.example.com
---
apiversion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: bar-host
spec:
  hostname: bar.example.com
  requestPolicy:
    insecure:
      action: Reject
```

- Since L4 load balancers cannot set `X-Forwarded-Protocol`, we don't use it at all here: instead, we dictate that 8080 and 8443 both speak cleartext HTTP, but everything arriving at port 8080 is insecure and everything at port 8443 is secure.

### Listening on Multiple Ports

There's no reason you need to use ports 8080 and 8443, or that you're limited to two ports. Here we'll use ports 9001 and 9002 for HTTP, and port 4001 for HTTPS. We'll route traffic irrespective of protocol.

```yaml
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorListener
metadata:
  name: listener-9001
spec:
  protocol: HTTP
  port: 9001
  securityModel: XFP
  hostBinding:     # This may well need editing for your case!
    namespace:
      from: SELF
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorListener
metadata:
  name: listener-9002
spec:
  protocol: HTTP
  port: 9002
  securityModel: XFP
  hostBinding:     # This may well need editing for your case!
    namespace:
      from: SELF
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorListener
metadata:
  name: listener-4001
spec:
  protocol: HTTPS
  port: 4001
  securityModel: XFP
  hostBinding:     # This may well need editing for your case!
    namespace:
      from: SELF
---
apiversion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: route-host
spec:
  requestPolicy:
    insecure:
      action: Route
```

- We can use `X-Forwarded-Proto` for all our `AmbassadorListener`s: the HTTP-only `AmbassadorListener`s will set it correctly.
- Each `AmbassadorListener` can specify only one port, but there's no hardcoded limit on the number of `AmbassadorListener`s you can have.

### Using Labels to Associate `AmbassadorHost`s and `AmbassadorListener`s

In the examples above, every `AmbassadorHost` is associated with every `AmbassadorListener`. You can use Kubernetes labels to restrict this association.

Here, we'll listen for HTTP to `foo.example.com` on port 8888, and for either HTTP or HTTPS to `bar.example.com` on port 9999 (where we'll redirect HTTP to HTTPS). Traffic to `baz.example.com` will work on both ports, and we'll route HTTP for it rather than redirecting.

```yaml
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorListener
metadata:
  name: listener-8888
spec:
  protocol: HTTP
  port: 8888
  securityModel: XFP
  hostBinding:
    selector:
      matchLabels:
        tenant: foo
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorListener
metadata:
  name: listener-9999
spec:
  protocol: HTTPS
  port: 9999
  securityModel: XFP
  hostBinding:
    selector:
      matchLabels:
        tenant: bar
---
apiversion: v1
kind: Secret
type: kubernetes.io/tls
metadata:
  name: bar-secret
data:
  tls.crt: -wildcard here-
  tls.key: -wildcard here-
---
apiversion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: foo-host
  labels:
    tenant: foo
spec:
  hostname: foo.example.com
  requestPolicy:
    insecure:
      action: Route
---
apiversion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: bar-host
  labels:
    tenant: bar
spec:
  hostname: bar.example.com
  tlsSecret:
    name: bar-secret
---
apiversion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: baz-host
  labels:
    tenant: foo
    tenant: bar
spec:
  hostname: baz.example.com
  tlsSecret:
    name: bar-secret
  requestPolicy:
    insecure:
      action: Route
```

- Note the `labels` on each `AmbassadorHost`, which the `hostBinding` on the `AmbassadorListener` can reference.
   - Note also that only label selectors are supported at the moment.

### Wildcard `AmbassadorHost`s and `AmbassadorMapping`s

In an `AmbassadorMapping`, the `host` is now treated as a glob rather than an exact match, with the goal of vastly reducing the need for `host_regex`. (The `hostname` in an `AmbassadorHost` has always been treated as a glob).

- **Note that only prefix and suffix matches are supported**, so `*.example.com` and `foo.*` are both fine, but `foo.*.com` will not work -- you'll need to use `host_regex` if you really need that. (This is an Envoy limitation.)

In this example, we'll accept both HTTP and HTTPS, but:

- Cleartext traffic to any host in `lowsec.example.com` will be routed.
- Cleartext traffic to any host in `normal.example.com` will be redirected.
- Any other cleartext traffic will be rejected.

```yaml
---
apiversion: v1
kind: Secret
type: kubernetes.io/tls
metadata:
  name: example-secret
data:
  tls.crt: -wildcard for *.example.com here-
  tls.key: -wildcard for *.example.com here-
---
apiversion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: lowsec-host
spec:
  hostname: "*.lowsec.example.com"
  tlsSecret:
    name: example-secret
  requestPolicy:
    insecure:
      action: Route
---
apiversion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: normal-host
spec:
  hostname: "*.normal.example.com"
  tlsSecret:
    name: example-secret
  requestPolicy:          # We could leave this out since
    insecure:             # redirecting is the default, but
      action: Redirect    # it's spelled out for clarity.
---
apiversion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: catchall-host
spec:
  hostname: "*.example.com"
  tlsSecret:
    name: example-secret
  requestPolicy:
    insecure:
      action: Reject
```

- We're relying on the default `AmbassadorListener`s here.
- The three `AmbassadorHost`s apply different insecure routing actions depending on the hostname.
- You could also do this with `host_regex`, but using `host` with globs will give better performance.
   - Being able to _not_ associate a given `AmbassadorMapping` with a given `AmbassadorHost` when the `AmbassadorMapping`'s `host` doesn't match helps a lot when you have many `AmbassadorHost`s.
   - Reliably determining if a regex (for the `AmbassadorMapping`) matches a glob (for the `AmbassadorHost`) isn't really possible, so we can't prune `host_regex` `AmbassadorMapping`s at all.
