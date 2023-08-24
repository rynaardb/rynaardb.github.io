---
layout: post
title: "10 Kubernetes Security Best Practices"
date: 2023-08-24
categories: devsecops kubernetes
tags: devsecops kubernetes security
image:
    path: /assets/images/security.webp
mermaid: true
---

Ensuring the security of your Kubernetes cluster is paramount to its efficient operation, the protection of sensitive data, and the resilience of your infrastructure. In this article, we'll explore ten crucial Kubernetes security best practices and provide guidelines on how to implement them.

## 1. Image Scanning for Vulnerabilities

Before deploying containers to your cluster, ensure that the images do not have known vulnerabilities.

How?

- You can use tools like Docker Scout or Snyk to scan images for vulnerabilities.
- You can also use the open-source tool Trivy to scan images for vulnerabilities in the base OS packages, application dependencies, and language-specific files.

Docker Scout:

```shell
docker scout cve <image-name>
```

Snyk:

```shell
snyk test <image-name>
```

Trivy:

```shell
trivy image <image-name>
```

## 2. Avoid Root User for Running Containers

Running containers as root can pose a security risk. If an attacker gains access to a container running as root, they can easily gain root access to the host machine.

How?

- Ensure that the containers are not running as root.
- You can do this by specifying a non-root user in the Dockerfile or the Kubernetes manifest by specifying the `runAsUser` security context.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: non-root-pod
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: non-root-container
    image: myimage
```

## 3. Users & Permissions using RBAC

Kubernetes uses Role-Based Access Control (RBAC) to control access to the cluster's resources. RBAC allows you to define granular permissions for users and groups.

How?

- Define roles and role bindings for users and groups
- Attach the roles to the users and groups

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: "foo"
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

> Apply least privilege when defining roles and role bindings. Only grant the permissions that are required for the user to perform their tasks.
{: .prompt-info }

## 4. Use Network Policies

Network policies allow you to control the traffic flow between pods and namespaces. They can be used to restrict access to pods and namespaces and prevent unauthorized access.

How?

- Define network policies to allow or deny traffic to pods and namespaces
- Attach the network policies to the pods and namespaces

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-foo
spec:
  podSelector:
    matchLabels:
      app: foo
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: bar
```

> Apply least privilege when defining network policies. Only allow the traffic that is required for the application to function.
{: .prompt-info }

## 5. Encrypt Communication

Encrypting communication between the Kubernetes components and the API server is essential to prevent unauthorized access to the cluster.

How?

- Enable TLS authentication for the API server
- Enable TLS authentication for the kubelet

## 6. Secure Secret Data

Secrets are used to store sensitive data like passwords, API keys, and certificates. It is essential to ensure that the secrets are stored securely and are not exposed to unauthorized users.

How?

- Use a secure secret storage backend like HashiCorp Vault

```shell
vault kv put secret/myapp/config username="foo" password="bar"
```

## 7. Secure etcd

etcd is a distributed key-value store that stores the cluster's state. It is essential to secure etcd to prevent unauthorized access to the cluster's state.

How?

- Enable TLS authentication for etcd
- Enable encryption for etcd

## 8. Backup & Disaster Recovery

It is essential to have a backup and disaster recovery plan in place to ensure business continuity in case of a disaster. This includes backing up the cluster's state and configuration and having a plan to recover from a disaster.

How?

- Ensure regular backups of the cluster's state and configuration are taken
- Have a disaster recovery plan in place
- Test the disaster recovery plan regularly

## 9. Configure Security Policies

Security policies allow you to define rules for the cluster's security. They can be used to enforce security best practices and prevent unauthorized access to the cluster. For example, you can define a policy to prevent users from running containers as root.

How?

- Define security policies to enforce security best practices
- Attach the security policies to the users and groups

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  ...
```

## 10. Limit Resource Usage

Limiting resource usage is essential to prevent resource exhaustion and Denial-of-Service (DoS) attacks. It is also important to ensure that the cluster is not over-provisioned.

How?

- Define resource limits for pods and namespaces
- Define resource quotas for pods and namespaces

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-limits-pod
spec:
  containers:
  - name: resource-limits-container
    image: myimage
    resources:
      limits:
        cpu: "1"
        memory: "1Gi"
      requests:
        cpu: "0.5"
        memory: "0.5Gi"
```

## Conclusion

By implementing these best practices, you will be taking proactive steps to ensure that your Kubernetes deployments are secure, efficient, and resilient against potential threats. Always stay updated with Kubernetes security best practices as the ecosystem and threat landscape evolve.
