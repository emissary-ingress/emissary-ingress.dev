---
title: Migration Matrix
---

<Alert severity="warning">
  <b>Read the instructions below</b> before making any changes to your cluster!
</Alert>

There are currently multiple paths for upgrading Emissary, depending on what version you're currently
running, what you want to be running, and whether you installed Emissary using [Helm](../helm) or
YAML.

(To check out if you installed Emissary using Helm, run `helm list --all` and see if
Emissary is listed. If so, you installed using Helm.)

<Alert severity="warning">
  <b>Read the instructions below</b> before making any changes to your cluster!
</Alert>

## If you are currently running Ambassador Edge Stack

See the [instructions on updating Ambassador Edge Stack](../../../../../edge-stack/$aesDocsVersion$/topics/install/migration-matrix).

## If you installed Emissary using Helm

| If you're running.               | You can upgrade to                                                                                                         |
|----------------------------------|----------------------------------------------------------------------------------------------------------------------------|
| Emissary $version$       | [Ambassador Edge Stack $aesVersion$](/docs/edge-stack/$aesDocsVersion$/topics/install/upgrade/helm/emissary-2.5/edge-stack-2.5) |
| Emissary 2.4.X           | [Emissary $version$](../upgrade/helm/emissary-2.4/emissary-2.5)                                                    |
| Emissary 2.0.5           | [Emissary $version$](../upgrade/helm/emissary-2.0/emissary-2.5)                                                    |
| Emissary 1.14.X          | [Emissary $version$](../upgrade/helm/emissary-1.14/emissary-2.5)                                                   |
| Emissary prior to 1.14.X | [Emissary 1.14.X](../../../../1.14/topics/install/upgrading)                                                       |

## If you installed Emissary manually by applying YAML

| If you're running.               | You can upgrade to                                                                                                         |
|----------------------------------|----------------------------------------------------------------------------------------------------------------------------|
| Emissary $version$       | [Ambassador Edge Stack $aesVersion$](/docs/edge-stack/$aesDocsVersion$/topics/install/upgrade/yaml/emissary-2.5/edge-stack-2.5) |
| Emissary 2.4.X           | [Emissary $version$](../upgrade/yaml/emissary-2.4/emissary-2.5)                                                    |
| Emissary 2.0.5           | [Emissary $version$](../upgrade/yaml/emissary-2.0/emissary-2.5)                                                    |
| Emissary 1.14.X          | [Emissary $version$](../upgrade/yaml/emissary-1.14/emissary-2.5)                                                   |
| Emissary prior to 1.14.X | [Emissary 1.14.X](../../../../1.14/topics/install/upgrading)                                                       |
