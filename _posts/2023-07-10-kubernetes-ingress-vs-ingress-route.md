---
layout: post
title: "Kubernetes Ingress vs. Ingress Route"
date: 2023-07-10
categories: devops kubernetes
tags: kubernetes ingress ingress-route
image:
    path: /assets/images/kubernetes.webp
mermaid: true
---

In Kubernetes, managing inbound traffic to applications running in a cluster is a crucial aspect of building a scalable and robust infrastructure. Kubernetes provides two primary options for handling incoming traffic: Ingress and Ingress Route. 

Although they serve similar purposes, they have some differences that make them suitable for different scenarios. In this blog post, we will explore the concepts of Ingress and Ingress Route, compare their features, and discuss when to use each one. So let's dive in!

## Ingress

Kubernetes Ingress is an API object that acts as an *entry point to the cluster for external traffic*. It provides a way to define rules for routing external requests to the appropriate services within the cluster. Ingress allows you to define multiple virtual hosts, SSL/TLS termination, and path-based routing using rules and annotations.

### Key Features

#### Load Balancing

Ingress distributes incoming traffic across backend services using load balancing algorithms like round-robin or session affinity.

#### Path-based Routing

Ingress allows you to route traffic based on the URL path. For example, you can configure `/api` to go to a different service than `/web`.

#### SSL/TLS Termination

Ingress supports terminating SSL/TLS connections and provides a way to configure certificates for secure communication with clients.

#### Name-based virtual Hosting

Ingress lets you define multiple virtual hosts, each with its own set of rules and backend services.

#### Fine-grained Routing Rules

Ingress supports annotations that allow you to add additional routing rules, rewrite paths, configure CORS, etc.

### Example

Let's take a look at an example Ingress definition:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /api
            backend:
              serviceName: api-service
              servicePort: 8080
          - path: /web
            backend:
              serviceName: web-service
              servicePort: 80
```
{: file="ingress.yaml" }

In this example, the Ingress routes requests for `example.com/api` to the `api-service` running on port `8080`, and requests for `example.com/web` to the `web-service` running on port `80`.

## Ingress Route

Ingress Route is a custom resource introduced by the Contour project, which extends the functionality of Kubernetes Ingress. It aims to provide a more powerful and flexible way of defining complex routing configurations within Kubernetes.

### Key Features

#### Declarative Configuration

Ingress Route uses a declarative configuration model, allowing you to define routes as Kubernetes resources, similar to other Kubernetes objects.

#### Enhanced Routing Capabilities

Ingress Route supports advanced routing features, such as header-based routing, regular expression matching, traffic mirroring, and more.

#### Enhanced TLS Management

Ingress Route simplifies TLS certificate management by integrating with Kubernetes Secrets, enabling automatic certificate provisioning and rotation.

#### Customizable Load Balancing

Ingress Route provides the ability to customize load balancing algorithms and configure advanced load balancing policies.

#### Envoy Proxy

Ingress Route relies on the Envoy Proxy as the underlying data plane, providing advanced features like circuit breaking, rate limiting, and observability.

### Example

Here's an example Ingress Route definition:

```yaml
apiVersion: contour.heptio.com/v1
kind: IngressRoute
metadata:
  name: my-ingress-route
spec:
  virtualhost:
    fqdn: example.com
  routes:
    - match: /api
      services:
        - name: api-service
          port: 8080
    - match: /web
      services:
        - name: web-service
          port: 80
```
{: file="ingress-route.yaml" }

In this example, the Ingress Route achieves the same routing behavior as the previous Ingress example. Requests for `example.com/api` are routed to `api-service` on port `8080`, and requests for `example.com/web` are routed to `web-service` on port `80`.

## Ingress vs. Ingress Route: When to use each?

Both Ingress and Ingress Route serve as entry points for routing traffic in Kubernetes, but they differ in terms of features and complexity. Here are some considerations to help you decide when to use each:

Use Ingress if:

- You need a straightforward way to define basic path-based routing and SSL/TLS termination.
- You prefer a standard Kubernetes API object without external dependencies.
- You want a simple and lightweight solution that meets your basic routing requirements.

Use Ingress Route if:

- You require advanced routing capabilities such as header-based routing, traffic mirroring, or regex matching.
- You need a declarative configuration model that integrates well with Kubernetes.
- You want to leverage features provided by Envoy Proxy as the underlying data plane.
- You have more complex routing scenarios and need fine-grained control over load balancing.

## Conclusion

Ingress and Ingress Route are two options available in Kubernetes for managing inbound traffic to your cluster. While Ingress provides a basic and straightforward way of routing traffic, Ingress Route offers advanced features and greater flexibility, making it suitable for more complex routing scenarios.

Choose Ingress for simpler use cases, and Ingress Route when you require enhanced routing capabilities and customization options. Ultimately, the choice depends on the specific needs of your application and infrastructure.
