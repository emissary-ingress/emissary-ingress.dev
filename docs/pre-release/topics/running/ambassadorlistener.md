# The `Listener` CRD

The `Listener` CRD defines where, and how, $productName$ should listen for requests from the network, and which `Host` definitions should be used to process those requests. For further examples of how to use `Listener`, see [Configuring $productName$ to Communicate](../../howtos/configure-communications).

```yaml
---
apiVersion: getambassador.io/v3alpha1
kind: Listener
metadata:
  name: example-listener
spec:
  port: 8080          # int32, port number on which to listen
  protocol: HTTPS     # HTTP, HTTPS, HTTPPROXY, HTTPSPROXY, TCP
  securityModel: XFP  # XFP (for X-Forwarded-Proto), SECURE, INSECURE
  l7Depth: 0          # int32
  hostBinding: 
    namespace:
      from: SELF      # SELF, ALL
    selector: ...     # Kubernetes label selector
```

| Element | Type | Definition |
| :------ | :--- | :--------- |
| `port` | `int32` | The network port on which $productName$ should listen. *Required.* |
| `protocol` | `enum`; see below | A high-level protocol type, like "HTTPS". *Exactly one of `protocol` and `protocolStack` must be supplied.* |
| `protocolStack` | array of `enum`; see below | A sequence of low-level protocols to layer together. *Exactly one of `protocol` and `protocolStack` must be supplied.* |
| `securityModel` | `enum`; see below | How does $productName$ decide whether requests here are secure? *Required.*|
| `l7Depth` | `int32` | How many layer 7 load balancers are between the edge of the network and $productName$? *Optional; default is 0.*|
| `hostBinding` | `struct`, see below | Mechanism for determining which `Host`s will be associated with this `Listener`. *Required* |

### `protocol` and `protocolStack`

`protocol` is the **recommended** way to tell $productName$ that a `Listener` expects connections using a well-known protocol. When using `protocol`, `protocolStack` may not also be supplied.

Valid `protocol` values are:

| `protocol` | Description |
| :--------- | :---------- |
| `HTTP` | Cleartext-only HTTP. HTTPS is not allowed. |
| `HTTPS` | Either HTTPS or HTTP -- Envoy's TLS support can tell whether or not TLS is in use, and set `X-Forwarded-Protocol` correctly. |
| `HTTPPROXY` | Cleartext-only HTTP, using the HAProxy `PROXY` protocol. |
| `HTTPSPROXY` | Either HTTPS or HTTP, using the HAProxy `PROXY` protocol. |
| `TCP` | TCP sessions without HTTP at all. You will need to use `TCPMapping`s to route requests for this `Listener`. |
| `TLS` | TLS sessions without HTTP at all. You will need to use `TCPMapping`s to route requests for this `Listener`. |

### `securityModel` and `l7Depth`

`securityModel` defines how the `Listener` will decide whether a request is secure or insecure:

| `securityModel` | Description |
| :--------- | :---------- |
| `SECURE` | Requests are always secure. You might set this if your load balancer always terminates TLS for you, and you can trust the clients. |
| `INSECURE` | Requests are always insecure. You might set this for an HTTP-only `Listener`, or a `Listener` for clients that are expected to be hostile. |
| `XFP` | Requests are secure if, and only if, `X-Forwarded-Proto` indicates HTTPS. |

- For many scenarios, `XFP` is a reasonable choice: it's easy to redirect HTTP requests to HTTPS, and easy to route the resulting HTTPS requests.
   - In normal usage, Envoy guarantees that `X-Forwarded-Proto` is set and reflects the actual wire protocol used when the request arrived at Envoy.
   - To accomodate layer 7 proxies in front of $productName$, set `l7Depth` to the number of L7 proxies through which a request must pass before arriving at Envoy.
- `SECURE` and `INSECURE` are helpful for cases where something downstream of $productName$ should allow only one kind of request to reach $productName$: for example, a `Listener` behind a load balancer that terminates TLS and checks client certificates might use `SecurityModel: SECURE`, then use `Host`s to reject insecure requests if one somehow arrives.

### `hostBinding`

`hostBinding` specifies how this `Listener` should determine which `Host`s are associated with it:

- `namespace.from` allows filtering `Host`s by the namespace of the `Host`:
   - `namespace.from: SELF` accepts only `Host`s in the same namespace as the `Listener`.
   - `namespace.from: ALL` accepts `Host`s in any namespace.
- `selector` accepts only `Host`s that has labels matching the selector.

`hostBinding` is mandatory, and at least one of `namespace.from` and `selector` must be set. If both are set, both must match for a `Host` to be accepted.

### `protocolStack`

**`protocolStack` is not recommended if you can instead use `protocol`.** 

Where `protocol` allows configuring the `Listener` to use well-known protocol stacks, `protocolStack` allows configuring exactly which protocols will be layered together. If `protocol` allows what you need, it is safer to use `Protocol` than to risk having the stack broken with an incorrect `protocolStack`.

The possible stack elements are:

| `ProtocolStack` Element | Description |
| :---------------------- | :---------- |
| `HTTP` | Cleartext-only HTTP; must be layered with `TLS` for HTTPS |
| `PROXY` | The HAProxy `PROXY` protocol |
| `TLS` | TLS |
| `TCP` | Raw TCP |

`protocolStack` supplies a list of these elements to describe the protocol stack. **Order matters.** Some examples:

| `protocolStack` | Description |
| :-------------- | :---------- |
| [ `HTTP`, `TCP` ] | Cleartext-only HTTP, exactly equivalent to `protocol: HTTP`. |
| [ `TLS`, `HTTP`, `TCP` ] | HTTPS or HTTP, exactly equivalent to `protocol: HTTPS`. |
| [ `PROXY`, `TLS`, `TCP` ] | The `PROXY` protocol, wrapping `TLS` _afterward_, wrapping raw TCP. This isn't equivalent to any `protocol` setting, and may be nonsensical. |

## Default `Listeners` 

### With TLS

If no `Listener`s are present in the system at all, but the system is otherwise configured in a way that permits TLS termination, two `Listener`s are created by default:

```yaml
---
apiVersion: getambassador.io/v3alpha1
kind: Listener
metadata:
  name: default-http
spec:
  port: 8080
  securityModel: XFP
  protocol: HTTPS
  l7Depth: 0
  hostBinding:
    namespace:
      from: SELF 
---
apiVersion: getambassador.io/v3alpha1
kind: Listener
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

This allows either HTTP or HTTPS on either port 8080 or port 8443, using `X-Forwarded-Proto` to determine whether a given request is secure or insecure.

### Without TLS

If no `Listener`s are present in the system at all, and the system is not configured in a way that permits TLS termination, only one `Listener` is created by default:

```yaml
---
apiVersion: getambassador.io/v3alpha1
kind: Listener
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
```

This allows cleartext HTTP only, on port 8080.

## Examples

### Using an L7 Load Balancer to Terminate TLS

In this scenario, a layer 7 load balancer ahead of Emissary will terminate TLS, so Emissary will always see HTTP with a known good `X-Forwarded-Protocol`, thus:

```yaml
---
apiversion: getambassador.io/v2
kind: Listener
metadata:
  name: lb-listener
spec:
  port: 8443
  protocol: HTTP
  securityModel: XFP  
  l7Depth: 1
  hostBinding: ...
```

- We specifically set this `Listener` to HTTP-only, but we stick with port 8443 just because we expect people setting up TLS at all to expect to use port 8443. (There's nothing special about the port number, pick whatever you like.)
- We set `securityModel` to XFP, because the L7 load balancer's `X-Forwarded-Proto` is valid, and `Host`s should be able to use it to make decisions about how to route.
- We set `l7Depth` to 1 to indicate that there's a single trusted L7 load balancer ahead of us.
- `hostBinding` isn't shown here, but it must be set correctly to find the `Host`s that should be associated with this `Listener`.

### Using a Split L4 Load Balancer to Terminate TLS

Here, we assume that Emissary is behind a load balancer setup that handles TLS at layer 4:

- Incoming cleartext traffic is forwarded to Emissary on port 8080.
- Incoming TLS traffic is terminated at the load balancer, then forwarded to Emissary *as cleartext* on port 8443.
- This might involve multiple L4 load balancers, but the actual number doesn't matter.
- The actual port numbers we use don't matter either, as long as Emissary and the load balancer(s) agree on which port is for which traffic.

```yaml
---
apiversion: getambassador.io/v2
kind: Listener
metadata:
  name: split-lb-one-listener
spec:
  protocol: HTTP
  port: 8080
  securityModel: INSECURE
---
apiversion: getambassador.io/v2
kind: Listener
metadata:
  name: split-lb-two-listener
spec:
  protocol: HTTP
  port: 8443
  securityModel: SECURE
```

- Since L4 load balancers cannot set `X-Forwarded-Protocol`, we don't use it at all here: instead, we dictate that 8080 and 8443 both speak cleartext HTTP, but everything arriving at port 8080 is insecure and everything at port 8443 is secure.
