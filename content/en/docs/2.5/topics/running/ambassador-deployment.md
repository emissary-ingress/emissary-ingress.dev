---
title: Deployment architecture
---

Emissary can be deployed in a variety of configurations. The specific configuration depends on your data center.

## Public cloud

If you're using a public cloud provider such as Amazon, Azure, or Google, Emissary can be deployed directly to a Kubernetes cluster running in the data center. Traffic is routed to Emissary via a cloud-managed load balancer such as an Amazon Elastic Load Balancer or Google Cloud Load Balancer. Typically, this load balancer is transparently managed by Kubernetes in the form of the `LoadBalancer` service type. Emissary then routes traffic to your services running in Kubernetes.

## On-Premise data center

In an on-premise data center, Emissary is deployed on the Kubernetes cluster. Instead of exposing it via the `LoadBalancer` service type, Emissary is exposed as a `NodePort`. Traffic is sent to a specific port on any of the nodes in the cluster, which route the traffic to Emissary, which then routes the traffic to your services running in Kubernetes. You'll also need to deploy a separate load balancer to route traffic from your core routers to Emissary. [MetalLB](https://metallb.universe.tf/) is an open-source external load balancer for Kubernetes designed for this problem. Other options are traditional TCP load balancers such as F5 or Citrix Netscaler.

## Hybrid data center

Many data centers include services that are running outside of Kubernetes on virtual machines. For Emissary to route to services both inside and outside of Kubernetes, it needs the real-time network location of all services. This problem is known as "[service discovery](https://www.datawire.io/guide/traffic/service-discovery-microservices/)" and Emissary supports using [Consul](https://www.consul.io). Services in your data center register themselves with Consul, and Emissary uses Consul-supplied data to dynamically route requests to available services.

## Hybrid on-premise data center

The diagram below details a common network architecture for a hybrid on-premise data center. Traffic flows from core routers to MetalLB, which routes to Emissary running in Kubernetes. Emissary routes traffic to individual services running on both Kubernetes and VMs. Consul tracks the real-time network location of the services, which Emissary uses to route to the given services.

![Architecture](../../../images/consul-ambassador.png)
