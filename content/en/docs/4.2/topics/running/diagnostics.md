---
title: Diagnostics
---

Emissary Diagnostics gives you a summary of the current status and Mappings of your cluster and its services.

## Troubleshooting

### Can't access Emissary Diagnostics Overview?

Create an Ambassador `Module` if one does not already exist, and add the following config to enable diagnostics data.

```yaml
apiVersion: getambassador.io/v3alpha1
kind: Module
metadata:
  name: ambassador
spec:
  config:
    diagnostics:
      enabled: true
```

### Still can't see Emissary Diagnostics?

Do a port forward on your Emissary pod

  ```shell
  # Namespace, deployment name, and pod url/port depend on your current install

  kubectl port-forward -n emissary deploy/emissary 8877
  ```

You will be able to access the diagnostics overview page by going to `http://localhost:8877/ambassador/v0/diag/`

### Emissary not routing your services as expected?

You will need to examine the logs and Emissary pod status. See [Debugging](../debugging) for more information.
