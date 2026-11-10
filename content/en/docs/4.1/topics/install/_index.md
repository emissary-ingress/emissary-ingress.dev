---
title: "Install"
description: "This section of the documentation provides installation instructions"
---

# Installing Emissary

## <img class="os-logo" src="../../images/helm-navy.png"/> Install with Helm

Helm, the package manager for Kubernetes, is the recommended way to install
Emissary. Full details are in the [quick start](../../quick-start/).

## <img class="os-logo" src="../../images/kubernetes.png"/> Install with Kubernetes YAML

Another way to install Emissary if you are unable to use Helm is to
directly apply Kubernetes YAML. Use `helm template` to render the
Helm chart into YAML, as described in the [quick start](../../quick-start/).

## Upgrade or migrate to a newer version

If you already have an existing installation of Ambassador Edge Stack or
Emissary, you can upgrade your instance. The [migration matrix](migration-matrix/)
shows you how.

## Container Images

Although our installation guides will favor using the `docker.io` container registry,
we publish Ambassador Edge Stack and Emissary releases to multiple registries.

Starting with version 1.0.0, you can pull the emissary image from any of the following registries:

- `docker.io/emissaryingress/`
- `gcr.io/datawire/`

We want to give you flexibility and independence from a hosting platform's uptime to support
your production needs for Ambassador Edge Stack or Emissary. Read more about
[Running Emissary in Production](../running).

# What’s Next?

Emissary has a comprehensive range of features to
support the requirements of any edge microservice. To learn more about how Emissary works, along with use cases, best practices, and more,
check out the [quick start](../../quick-start/) or read the [Emissary
Story](../../about/why-ambassador).
