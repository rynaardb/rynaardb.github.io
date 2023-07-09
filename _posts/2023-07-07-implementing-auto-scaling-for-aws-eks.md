---
layout: post
title: "Implementing Auto-Scaling for AWS EKS"
date: 2023-07-07
categories: devops aws kubernetes
tags: aws eks kubernetes cluster-autoscaler terraform helm
image:
    path: /assets/images/aws-eks.webp
---

In the age of cloud computing, setting up and maintaining an efficient, scalable infrastructure is crucial for the success of any application. Tools like Kubernetes and Terraform help us manage complex infrastructures and ensure our services are reliable and resilient.

In this post, we'll be focusing on [AWS EKS (Elastic Kubernetes Service)](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html), a managed Kubernetes service offered by Amazon Web Services (AWS), and [Terraform](https://www.terraform.io), an Infrastructure as Code (IaC) tool by HashiCorp. Specifically, we will explore how to set up an auto-scaling feature for AWS EKS using Terraform. This will involve creating an EKS cluster, implementing the Kubernetes Cluster Autoscaler, and automating the process with Terraform.

## Prerequisites

Before proceeding, you should have:

- An AWS account
- Basic understanding of Kubernetes
- Basic understanding of Terraform
- AWS CLI installed and configured
- kubectl installed
- Helm installed
- Terraform installed

## What is AWS EKS?

AWS Elastic Kubernetes Service (EKS) is a managed container service that makes it easier for you to run, scale, and monitor your applications using Kubernetes, an open-source container orchestration platform.

Kubernetes allows you to deploy and manage multi-container applications at scale. It organizes containers into Pods (a group of one or more containers) and ensures they are correctly scheduled and run on the appropriate physical or virtual machines in your environment. It also monitors the health of these containers and schedules replacements if a container fails.

### Control Plane

This is the heart of an EKS cluster, consisting of several Kubernetes master nodes that maintain the desired state of your cluster, such as which applications are running or which container images to use. AWS EKS fully manages this control plane, relieving you from the overhead of setting up, scaling, and updating the Kubernetes masters.

### Worker Nodes

These are the EC2 instances that run your applications. While the EKS control plane is fully managed by AWS, you have full control over the worker nodes, i.e., you can choose the right types and number of instances to meet your application’s needs.

### Pods and Services

Applications running on EKS are packaged into Pods. A Pod represents a single instance of your application and can consist of one or multiple containers. Pods are ephemeral, i.e., they might be terminated and recreated dynamically. Kubernetes Services provide stable endpoints to interact with these Pods.

### Load Balancing

Kubernetes Services provide load balancing capabilities for routing traffic to Pods. You can also use AWS Load Balancers (ELB, NLB, or ALB) with EKS for more advanced load balancing capabilities.

### Auto Scaling

EKS integrates with the Kubernetes Cluster Autoscaler and the AWS Auto Scaling groups, allowing your applications to scale in or out as needed, improving your application's availability and reducing costs.

### Security

EKS works with AWS IAM (Identity and Access Management) for fine-grained access control to your Kubernetes resources. It also supports private cluster endpoints, allowing you to access the Kubernetes API server from your VPC only.

### Integration with AWS Services

EKS works seamlessly with AWS services like RDS (for databases), S3 (for object storage), IAM (for authentication and authorization), CloudWatch (for logs and metrics), and many others, giving you the flexibility to choose the right tools for your applications.

### Monitoring and Diagnostics

EKS integrates with AWS CloudWatch and AWS X-Ray, allowing you to monitor your applications, collect and analyze operational data, and get a comprehensive view of your applications' performance.

## EKS Cluster Setup with Terraform

### Initialize the Terraform Project

First, we'll initialize our Terraform project by creating a new directory and an empty `main.tf` file. The `main.tf` file will contain the Terraform configurations.

```shell
mkdir eks-autoscaling && cd eks-autoscaling && touch main.tf
```

### Define the AWS Provider

In `main.tf`, define the AWS provider. Here we're specifying the AWS region to `us-west-2`. Update it according to your needs.

```hcl
provider "aws" {
  region = "us-west-2"
}
```
{: file="main.tf" }

### Create the EKS Cluster

Use the `aws_eks_cluster` resource to create the EKS cluster. We'll create a two-node cluster for simplicity, but you can increase the number of nodes based on your needs.

```hcl
module "eks" {
  source          = "terraform-aws-modules/eks/aws"
  cluster_name    = "my-eks-cluster"
  cluster_version = "1.27"
  subnet_ids      = ["subnet-abcde012", "subnet-bcde012a", "subnet-fghi345a"]
  vpc_id          = "vpc-abcde012"

  eks_managed_node_group_defaults = {
    instance_types = ["t3.micro"]
    capacity_type  = "SPOT"
    disk_size      = 10
  }
}
```
{: file="main.tf" }

> In the `subnet_ids` attribute, specify the IDs of your VPC subnets where you want your EKS nodes to reside. The `vpc_id` is the ID of your AWS VPC. Modify the `key_name` attribute to the name of your AWS key pair.
{: .prompt-info }

### Apply the Configuration

```shell
terraform init
terraform apply
```

When asked for confirmation, type `yes`. Terraform will then proceed to create the EKS cluster.

> Use `terraform apply -auto-approve` to automatically approve the changes and to avoid being prompted for confirmation before proceeding. This can be useful while testing, but be careful when doing this in a production environment.
{: .prompt-tip }

## Implement Kubernetes Cluster Autoscaler

Once our EKS cluster is ready, the next step is to implement the Kubernetes Cluster Autoscaler, which adjusts the size of the Kubernetes cluster based on the current load.

### Add Helm Repository

To install the Cluster Autoscaler, we'll use a Helm chart. First, add the `autoscaler` Helm repository:

```shell
helm repo add autoscaler https://kubernetes.github.io/autoscaler
helm repo update
```

### Install the Cluster Autoscaler Helm Chart

Use Helm to install the Cluster Autoscaler on your EKS cluster:

```shell
$ helm install cluster-autoscaler autoscaler/cluster-autoscaler \
  --namespace kube-system \
  --set autoDiscovery.clusterName=my-eks-cluster \
  --set awsRegion=us-west-2 \
  --set rbac.create=true \
  --set rbac.serviceAccount.create=true \
  --set rbac.serviceAccount.annotations."eks\.amazonaws\.com/role-arn"=arn:aws:iam::123456789012:role/my-iam-role
```

> Replace `my-eks-cluster` with your cluster name, `us-west-2` with your AWS region, and the AWS role ARN with the ARN of the IAM role for your EKS cluster.
{: .prompt-info }

The Cluster Autoscaler will now monitor the pods in your cluster and scale the cluster up or down as needed.

## Automating the Process with Terraform

Lastly, we'll automate the above steps with Terraform. This involves creating a Helm chart resource in our `main.tf` file.

### Define the Helm Provider

In `main.tf`, define the Helm provider and specify the `kubernetes` provider:

```hcl
provider "helm" {
  kubernetes {
    config_path = "~/.kube/config"
  }
}
```
{: file="main.tf" }

### Create the Cluster Autoscaler Helm Chart Resource

Next, use the `helm_release` resource to create the Cluster Autoscaler Helm chart:

```hcl
resource "helm_release" "cluster_autoscaler" {
  name       = "cluster-autoscaler"
  namespace  = "kube-system"
  repository = "https://kubernetes.github.io/autoscaler"
  chart      = "cluster-autoscaler"
  version    = "9.9.2"

  set {
    name  = "autoDiscovery.clusterName"
    value = "my-eks-cluster"
  }

  set {
    name  = "awsRegion"
    value = "us-west-2"
  }

  set {
    name  = "rbac.create"
    value = "true"
  }

  set {
    name  = "rbac.serviceAccount.create"
    value = "true"
  }

  set {
    name  = "rbac.serviceAccount.annotations.eks\\.amazonaws\\.com/role-arn"
    value = "arn:aws:iam::123456789012:role/my-iam-role"
  }
}
```

> Replace the values as necessary.
{: .prompt-info }

### Apply the Configuration

Finally, apply the updated Terraform configuration:

```shell
terraform apply
```

> Creating the resources on AWS will cost you money. Remember to remove them once you are done to avoid unnecessary costs by doing a `terraform destroy`.
{: .prompt-warning }

## Conclusion

In this blog post, we have discussed how to set up an auto-scaling feature for AWS EKS using Terraform. We first created an EKS cluster, then implemented the Kubernetes Cluster Autoscaler, and finally automated the process with Terraform. This approach allows your applications to scale automatically based on demand, ensuring efficient resource utilization and cost savings.

Please note that this post provides a basic example of auto-scaling. For a production environment, you should carefully design your auto-scaling strategy considering factors such as cost, performance, and fault tolerance. In addition, remember to secure your cluster following AWS and Kubernetes best practices.
