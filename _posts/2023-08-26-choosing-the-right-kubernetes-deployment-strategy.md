---
layout: post
title: "Choosing the Right Kubernetes Deployment Strategy"
date: 2023-08-26
categories: devops kubernetes
tags: devops kubernetes deployments
image:
    path: /assets/images/kubernetes.webp
---

In the world of containerized applications, deploying and managing updates efficiently is crucial. Kubernetes, with its robust orchestration capabilities, offers a variety of deployment strategies to achieve this. Each strategy has its own strengths and use cases.

In this post, we'll dive into six popular Kubernetes deployment strategies: Rolling Update, Recreate, Blue/Green, Canary, Shadow, and A/B Testing. We'll explore what each strategy entails, their pros and cons, and provide code examples to illustrate their implementation.

## Rolling Update

The Rolling Update strategy is designed to minimize downtime during application updates. It gradually replaces instances of the old version with instances of the new version. This ensures that the application remains available to users throughout the deployment process.

**Pros**

- Minimal downtime, ensuring high availability during updates.
- Gradual replacement of old pods with new ones, reducing risk.
- Ability to roll back to the previous version if issues arise.

**Cons**

- Complex management of inter-version dependencies might be required.
- Slightly slower rollout due to the gradual replacement process.

**Use Cases**

- Applications that require high availability and minimal downtime.
- Applications with complex inter-version dependencies.

**Example**

In your Kubernetes Deployment manifest, set the `strategy` field to `RollingUpdate` and specify the `maxSurge` and `maxUnavailable` values:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v2
```

In this example, one old pod is replaced with a new pod at a time (`maxSurge: 1`) while ensuring there's always at least one pod available (`maxUnavailable: 1`).

{: .prompt-info }
> If you don't specify a deployment strategy, Kubernetes defaults to Rolling Update with `maxSurge: 25%` and `maxUnavailable: 25%`.

## Recreate

The Recreate strategy is straightforward but involves some downtime. It involves terminating all instances of the old version before deploying the new version. This strategy is suitable for applications where downtime is acceptable.

**Pros**

- Simple and straightforward deployment process.
- Ensures a clean slate with no old instances, reducing potential conflicts.

**Cons**

- Involves downtime during the update process, impacting availability.
- Doesn't allow for gradual transition, which might lead to performance dips.

**Use Cases**

- Applications that can tolerate downtime during updates.
- Limited resrouces that can't support multiple versions simultaneously.
- Development environments where availability isn't a concern.

**Example**

In your Deployment manifest, set the `strategy` field to `Recreate`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v2
```

This strategy doesn't require additional configuration as it terminates all old pods before creating new ones.

## Blue/Green

The Blue/Green strategy maintains two identical environments: one running the current version (Blue) and another for the new version (Green). The switch from Blue to Green is done instantly, minimizing downtime and enabling easy rollback.

**Pros**

- Instant switch between versions, minimizing downtime and improving availability.
- Easy rollback to the previous version by switching back to the original environment.
- Enables testing and validation of the new version before routing all traffic.

**Cons**

- Requires maintaining two identical environments, potentially doubling resource usage.
- Complex routing configuration for production traffic switching.

**Use Cases**

- Applications that require high availability and minimal downtime.
- Testing and validation of new versions before full rollout.

**Example**

Utilize Kubernetes Services and Ingress resources to manage the switch between Blue and Green deployments.

Deploy Blue and Green versions with different labels:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp-blue
      replica: blue
  template:
    metadata:
      labels:
        app: myapp-blue
        replica: blue
    spec:
      containers:
      - name: myapp
        image: myapp:v1
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp-green
      replica: green
  template:
    metadata:
      labels:
        app: myapp-green
        replica: green
    spec:
      containers:
      - name: myapp
        image: myapp:v2
```

Create a Service and Ingress for Blue and Green:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-blue
spec:
  selector:
    app: myapp-blue
    replica: blue
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-green
spec:
  selector:
    app: myapp-green
    replica: green
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp
spec:
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-blue
            port:
              number: 80
```

To switch from Blue to Green, update the Service selector:

```shell
kubectl apply -f my-app-blue.yaml
kubectl apply -f my-app-green.yaml
```

## Canary

The Canary strategy involves deploying the new version to a subset of users or traffic. This allows for controlled testing before rolling out the update to the entire user base.

**Pros**

- Controlled testing of new features with a subset of users or traffic.
- Early detection of issues in the new version while limiting potential impact.
- Smooth transition to a full rollout if the Canary phase is successful.

**Cons**

- Requires additional configuration for traffic splitting and monitoring.
- Might not be suitable for all types of applications, especially those with heavy inter-component dependencies.

**Use Cases**

- Validation of new features with a subset of users or traffic.
- Early detection of issues in the new version while limiting potential impact.

**Example**

Use Kubernetes Service and Weighted Pods to implement the Canary deployment.

Deploy the Canary version with a specific label:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp-canary
  template:
    metadata:
      labels:
        app: myapp-canary
    spec:
      containers:
      - name: myapp
        image: myapp:v2
```

Create a Service for Canary deploymet:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-canary
spec:
  selector:
    app: myapp-canary
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

Implement traffic splitting with Istio (assuming Istio is installed):

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp.com
  http:
  - route:
    - destination:
        host: myapp-canary
        subset: v2
      weight: 10
    - destination:
        host: myapp
        subset: v1
      weight: 90
```

Gradually increase the Canary weight and observe the behavior of the new version.

## Shadow

The Shadow strategy involves sending a copy of production traffic to the new version without affecting the response. This helps in observing the behavior of the new version in real-world scenarios.

**Pros**

- Real-time observation of new version behavior in a production environment.
- No impact on actual user experience since shadow traffic is separate.
- Useful for gauging performance metrics and validating changes.

**Cons**

- Requires careful configuration to ensure that shadow traffic doesn't affect metrics.
- Complex setup involving traffic splitting and monitoring.

**Use Cases**

- Real-time observation of new version behavior in a production environment.
- Validation of performance metrics and changes.

**Example**

Deploy the Shadow version with a specific label:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-shadow
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp-shadow
  template:
    metadata:
      labels:
        app: myapp-shadow
    spec:
      containers:
      - name: myapp
        image: myapp:v2
```

Create a Service for the Shadow deployment:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-shadow
spec:
  selector:
    app: myapp-shadow
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

Implement traffic splitting with Istio (assuming Istio is installed):

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp.com
  http:
  - route:
    - destination:
        host: myapp-shadow
        subset: v2
      weight: 100
    - destination:
        host: myapp
        subset: v1
      weight: 0
```

## A/B Testing

The A/B Testing strategy involves running multiple versions simultaneously and directing a portion of the traffic to each version. This allows for comparative analysis of different versions.

**Pros**

- Comparative analysis of two versions to assess user response and performance.
- Flexibility to allocate different portions of traffic to each version.
- Enables data-driven decision-making for version selection.

**Cons**

- Requires continuous monitoring to draw accurate conclusions from the A/B test.
- Complex traffic splitting and analysis tools might be needed.

**Use Cases**

- Comparative analysis of two versions to assess user response and performance.
- Data-driven decision-making for version selection.

**Example**

Deploy both versions with different labels:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp-v1
  template:
    metadata:
      labels:
        app: myapp-v1
    spec:
      containers:
      - name: myapp
        image: myapp:v1
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-v2
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp-v2
  template:
    metadata:
      labels:
        app: myapp-v2
    spec:
      containers:
      - name: myapp
        image: myapp:v2
```

Create Services for both deployments:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-v1
spec:
  selector:
    app: myapp-v1
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-v2
spec:
  selector:
    app: myapp-v2
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

## Conclusion

Kubernetes provides a rich set of deployment strategies, each tailored to different application requirements and scenarios. From the seamless updates of Rolling Updates to the controlled testing of Canary deployments, and the instant switching of Blue/Green, these strategies empower developers to manage the lifecycles of their applications with flexibility and confidence.

Whether you prioritize zero downtime, rapid iteration, risk mitigation, or comparative analysis, Kubernetes deployment strategies offer a toolbox of options to meet your deployment needs. By understanding and implementing these strategies effectively, you can ensure smoother application updates and improved user experiences, all within the dynamic and ever-evolving landscape of containerized applications.
