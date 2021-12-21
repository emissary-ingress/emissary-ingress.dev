import Alert from '@material-ui/lab/Alert';
import './index.less'

# Installing $productName$

<div class="docs-article-toc">
<h3>Contents</h3>

* [Install with Helm](#img-classos-logo-srcimageshelm-navypng-install-via-helm)
* [Install with Kubernetes YAML](#img-classos-logo-srcimageskubernetespng-install-via-kubernetes-yaml)
* [Try the demo with Docker](#img-classos-logo-srcimagesdockerpng-install-locally-on-docker)
* [Upgrade or migrate to a newer version](#upgrade-options)
* [Container Images](#container-images)
* [What's Next?](#whats-next)

</div>

## <img class="os-logo" src="../../images/helm-navy.png"/> Install with Helm
Helm, the package manager for Kubernetes, is the recommended way to install
$productName$. Full details are in the [Helm instructions.](helm/)

## <img class="os-logo" src="../../images/kubernetes.png"/> Install with Kubernetes YAML
Another way to install $productName$ if you are unable to use Helm is to
directly apply Kubernetes YAML. See details in the
[manual YAML installation instructions.](yaml-install).

## <img class="os-logo" src="../../images/docker.png"/> Try the demo with Docker
The Docker install will let you try the $productName$ locally in seconds,
but is not supported for production workloads. [Try $productName$ on Docker.](docker/)

## Upgrade or migrate to a newer version
If you already have an existing installation of $AESproductName$ or
$OSSproductName$, you can upgrade your instance. The [migration matrix](migration-matrix/)
shows you how.

## Container Images
Although our installation guides will favor using the `docker.io` container registry,
we publish $AESproductName$ and $OSSproductName$ releases to multiple registries.

Starting with version 1.0.0, you can pull the $productDockerImage$ image from any of the following registries:
- `docker.io/emissaryingress/`
- `gcr.io/datawire/`

We want to give you flexibility and independence from a hosting platform's uptime to support
your production needs for $AESproductName$ or $OSSproductName$. Read more about
[Running $productName$ in Production](../running).

# What’s Next?
$productName$ has a comprehensive range of [features](/features/) to
support the requirements of any edge microservice. To learn more about how $productName$ works, along with use cases, best practices, and more,
check out the [Welcome page](../../) or read the [$productName$
Story](../../about/why-ambassador).
