---
title: Migration Matrix
weight: -10
---

<Alert severity="warning">
  <b>Read the instructions below</b> before making any changes to your cluster!
</Alert>

There are currently multiple paths for upgrading Emissary, depending on what version you're currently
running, what you want to be running, and whether you installed Emissary using Helm or YAML.

(To check out if you installed Emissary using Helm, run `helm list --all-namespaces` and see if
Emissary is listed. If so, you installed using Helm.)

<Alert severity="warning">
  <b>Read the instructions below</b> before making any changes to your cluster!
</Alert>

## If you installed Emissary using Helm

| If you're running.     | You can upgrade to                                         |
|------------------------|------------------------------------------------------------|
| Emissary 3.10          | [Emissary 4.2](upgrade/helm/emissary-3.10/)                |
| Emissary 3.9           | [Emissary 3.10](upgrade/helm/emissary-3.9/)                |
| Emissary 3.8.X         | [Emissary 3.10](upgrade/helm/emissary-3.8)                 |
| Emissary 3.7.X         | [Emissary 3.10](upgrade/helm/emissary-3.7)                 |
| Emissary 2.5           | [Emissary 3.10](upgrade/helm/emissary-2.5)                 |
| Emissary 2.4 or earlier | See the [3.10 migration matrix](/docs/3.10/topics/install/migration-matrix/) |

## If you installed Emissary manually by applying YAML

| If you're running.     | You can upgrade to                                         |
|------------------------|------------------------------------------------------------|
| Emissary 3.10          | [Emissary 4.2](upgrade/yaml/emissary-3.10/)                |
| Emissary 3.9           | [Emissary 3.10](upgrade/yaml/emissary-3.9/)                |
| Emissary 3.8.X         | [Emissary 3.10](upgrade/yaml/emissary-3.8/)                |
| Emissary 3.7.X         | [Emissary 3.10](upgrade/yaml/emissary-3.7/)                |
| Emissary 2.5           | [Emissary 3.10](upgrade/yaml/emissary-2.5/)                |
| Emissary 2.4 or earlier | See the [3.10 migration matrix](/docs/3.10/topics/install/migration-matrix/) |
