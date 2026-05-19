---
title: Distributed Tracing with OpenTelemetry and Dash0
---

In this tutorial, we'll configure Emissary to initiate a trace on some sample requests, collect them with the OpenTelemetry Collector and use Dash0 to visualize them.

## Before you get started

This tutorial will walk you through setting up Emissary with distributed tracing using Dash0. We'll use a local [kind](https://kind.sigs.k8s.io/) cluster for this guide.

First, create a kind cluster:

```bash
kind create cluster --name emissary
```

Then follow the [Quick Start](../../quick-start) guide to install Emissary and the Faces demo application. After completing those steps, you'll have a Kubernetes cluster running Emissary and the Faces demo. Now let's add distributed tracing to this setup.

## 1. Setup Dash0

If you don't already have a Dash0 account, sign up at [dash0.com](https://www.dash0.com/) for a 14-day free trial. Once logged in, you'll need to obtain your authentication token and endpoint information from your Dash0 organization settings. Save the following information:

- **Auth Token**: Your Dash0 authorization token
- **Endpoint**: Your Dash0 OTLP gRPC endpoint (hostname and port)

## 2. Deploy the OpenTelemetry Collector

The next step is to deploy the OpenTelemetry Collector. The purpose of the OpenTelemetry Collector is to receive trace data from Emissary and export it to Dash0.

For the purposes of this tutorial, we are going to create and use the `opentelemetry` namespace. This can be done with the following command:

```bash
kubectl create namespace opentelemetry
```

Next, we need to create a Kubernetes secret to store your Dash0 credentials. Replace the placeholder values with your actual Dash0 credentials:

```bash
kubectl create secret generic dash0-secrets \
  --namespace opentelemetry \
  --from-literal=dash0-authorization-token='YOUR_DASH0_AUTH_TOKEN' \
  --from-literal=dash0-grpc-hostname='YOUR_DASH0_GRPC_HOSTNAME' \
  --from-literal=dash0-grpc-port='4317'
```

Now we'll deploy the OpenTelemetry Collector using Helm. First, add the OpenTelemetry Helm repository:

```bash
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo update
```

Create a values file named `otel-collector-values.yaml` with the following configuration:

```yaml
mode: deployment
replicaCount: 1
image:
  repository: otel/opentelemetry-collector-contrib

resources:
  limits:
    memory: 512Mi
  requests:
    memory: 256Mi

extraEnvs:
  - name: DASH0_AUTH_TOKEN
    valueFrom:
      secretKeyRef:
        name: dash0-secrets
        key: dash0-authorization-token
  - name: DASH0_ENDPOINT_OTLP_GRPC_HOSTNAME
    valueFrom:
      secretKeyRef:
        name: dash0-secrets
        key: dash0-grpc-hostname
  - name: DASH0_ENDPOINT_OTLP_GRPC_PORT
    valueFrom:
      secretKeyRef:
        name: dash0-secrets
        key: dash0-grpc-port

config:
  receivers:
    otlp:
      protocols:
        grpc:
          endpoint: 0.0.0.0:4317

  exporters:
    otlp/dash0:
      auth:
        authenticator: bearertokenauth/dash0
      endpoint: ${env:DASH0_ENDPOINT_OTLP_GRPC_HOSTNAME}:${env:DASH0_ENDPOINT_OTLP_GRPC_PORT}

  extensions:
    bearertokenauth/dash0:
      scheme: Bearer
      token: ${env:DASH0_AUTH_TOKEN}

  service:
    extensions:
      - bearertokenauth/dash0
      - health_check
    pipelines:
      traces:
        receivers:
          - otlp
        exporters:
          - otlp/dash0
```

Deploy the OpenTelemetry Collector with Helm:

```bash
helm install otel-collector-opentelemetry-collector \
  open-telemetry/opentelemetry-collector \
  --namespace opentelemetry \
  --values otel-collector-values.yaml
```

At this point, the OpenTelemetry Collector should be setup properly and ready to send data to Dash0.

## 3. Configure the TracingService

Now that the OpenTelemetry Collector is setup for collecting data, the next step will be to configure our [TracingService](../../topics/running/services/tracing-service). We will be using the OpenTelemetry driver to send trace data to the OpenTelemetry Collector. Please apply the following YAML:

```yaml
---
apiVersion: getambassador.io/v3alpha1
kind: TracingService
metadata:
  name: otel-tracing
  namespace: emissary
spec:
  service: "otel-collector-opentelemetry-collector.opentelemetry.svc.cluster.local:4317"
  driver: opentelemetry
  config:
    service_name: emissary-ingress
  sampling:
    overall: 100
```

As a final step we want to restart Emissary as this is necessary to add the distributed tracing headers. This command will restart all the Pods (assuming Emissary is installed in the <code>emissary</code> namespace):

```bash
kubectl -n emissary rollout restart deploy
```

<Alert severity="warning">
  Restarting Emissary is required after deploying a Tracing Service for changes to take effect.
</Alert>

## 4. Testing Distributed Tracing

Finally, we are going to test our distributed tracing. This tutorial assumes you have completed the [Quick Start](../../quick-start) guide and have the Faces demo application installed.

First, set up port-forwarding to access Emissary:

```bash
kubectl port-forward -n emissary svc/emissary-emissary-ingress 8080:80
```

Now open `http://localhost:8080/faces/` in your browser. The Faces demo application will automatically generate requests, and with our sampling configuration set to 100%, all requests will be traced.

At this point, you should be able to view and check your traces in the Dash0 application. Log in to your Dash0 account and navigate to the traces view to see the distributed traces from your Emissary deployment.

## More

For a complete working example with all configurations, see the [Dash0 Emissary ingress example](https://github.com/dash0hq/dash0-examples/tree/main/emissary-ingress) and the [Dash0 Integration](https://www.dash0.com/hub/integrations/int_emissary_ingress/overview).
