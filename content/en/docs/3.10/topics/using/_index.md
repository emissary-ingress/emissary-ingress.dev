---
title: "Using"
description: "This section of the documentation provides instructions on using Emissary"
---

# Using Emissary

Application development teams use Emissary to manage edge policies associated with a specific service. This section of the documentation covers core Emissary elements that are typically used by the application development team.

* [Introduction to Mappings](intro-mappings) The `Mapping` resource is the core resource used by every application development team.
* Mapping Configuration:
  * [Automatic Retries](retries)
  * [Canary Releases](canary)
  * [Circuit Breakers](circuit-breakers)
  * [Cross Origin Resource Sharing](cors)
  * HTTP Headers
    * [Header-based Routing](headers/headers)
    * [Host Header](headers/host)
    * [Adding Request Headers](headers/add-request-headers)
    * [Adding Response Headers](headers/add-response-headers)
    * [Removing Request Headers](headers/remove-request-headers)
    * [Remove Response Headers](headers/remove-response-headers)
  * [Query Parameter Based Routing](query-parameters)
  * [Keepalive](keepalive)
  * Protocols
    * [TCP](tcpmappings)
    * gRPC, HTTP/1.0, gRPC-Web, WebSockets
  * [RegEx-based Routing](prefix-regex)
  * [Redirects](redirects)
  * [Rewrites](rewrites)
  * [Timeouts](timeouts)
  * [Traffic Shadowing](shadowing)
* [Advanced Mapping Configuration](mappings)
* Rate Limiting
  * [Introduction to Rate Limits](rate-limits/)
* [Gateway API](gateway-api)
