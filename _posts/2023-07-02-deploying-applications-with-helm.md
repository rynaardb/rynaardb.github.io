---
layout: post
title: "Deploying applications with Helm - A powerful Kubernetes Package Manager"
date: 2023-07-02
categories: devops helm
tags: helm helm-charts kubernetes package-manager
image:
    path: /assets/images/helm.webp
---

Kubernetes has become the de facto standard for container orchestration, providing a powerful platform for deploying and managing applications. However, managing the complexity of deploying multiple services and managing their configurations can be a daunting task.

This is where [Helm](https://helm.sh), a package manager for Kubernetes, comes to the rescue. Helm simplifies the process of deploying applications by providing a convenient way to define, install, and upgrade applications in a consistent manner.

## What is Helm?

Helm is a tool that streamlines the installation and management of applications on Kubernetes clusters. It provides a convenient way to define, install, and upgrade applications using a concept called "charts." A Helm chart is a collection of files that describe the configuration, resources, and dependencies of a Kubernetes application. Helm manages the lifecycle of a chart, making it easy to install, upgrade, and remove applications.

## Installation

Before we dive into using Helm, let's first install it on our local machine or the server where you want to manage your Kubernetes applications. Helm is available for various operating systems, including Linux, macOS, and Windows. Follow the official Helm installation guide for detailed instructions on how to install Helm on your desired platform.

## Helm Charts

A Helm chart is a collection of files that describe a set of Kubernetes resources necessary to run an application. It contains templates, configuration values, and optional dependencies. Helm charts follow a predefined directory structure and can be easily shared and versioned using a chart repository.

### Anatomy of a Helm Chart

A Helm chart consists of the following key components:

**Chart.yaml**

This file contains metadata about the chart, such as its name, version, description, and maintainer information.

**values.yaml**

This file defines the default configuration values for the chart. Users can override these values during installation or upgrade.

**templates/**

This directory contains the Kubernetes manifests and other template files used to generate the actual resources during installation.

**charts/**

This directory can contain sub-charts or dependencies required by the main chart.

Here's an example of a basic Helm chart directory structure:

```markdown
myapp/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── charts/
    └── redis-7.0.2/
        ├── Chart.yaml
        └── values.yaml
```

> Run the `helm create myapp` command to create a chart directory along with the common files and directories used in a chart.
{: .prompt-tip }

### Chart Dependencies

Helm charts can have dependencies on other charts, allowing you to compose complex applications from reusable components. Dependencies are defined in the Chart.yaml file using the dependencies section. Helm fetches and installs the dependencies automatically when you install or upgrade a chart.

Here's an example of how to define a dependency in `Chart.yaml`:

```yaml
dependencies:
  - name: redis
    version: 7.0.2
    repository: https://example.com/charts
```
{: file="Chart.yaml" }

### Customizing Chart Values

One of the key features of Helm is the ability to customize chart values during installation or upgrade. The `values.yaml` file defines the default values for the chart, which can be overridden by providing a separate value file or using command-line flags.

For example, you can create a `myvalues.yaml` file with custom values:

```yaml
replicaCount: 3
image:
  repository: myapp
  tag: 1.0.0
```
{: file="myvalues.yaml" }

To install the chart with these custom values, you can use the following command:

```shell
helm install myapp ./myapp -f myvalues.yaml
```

## Working with Helm

With a Helm chart in place, let's see how to use Helm to deploy and manage applications on Kubernetes clusters.

### Installing a Chart

To install a Helm chart, you can use the `helm install` command followed by the name you want to give to the release and the chart's directory or package location:

```shell
helm install myapp ./myapp
```

This command installs the chart with the release name "myapp" using the chart located in the ./myapp directory.

### Upgrading a Chart

When you need to update an application deployed with Helm, you can use the `helm upgrade` command. This command takes the release name and the chart's directory or package location as arguments:

```shell
helm upgrade myapp ./myapp
```

### Uninstalling a Chart

To remove an application and all associated resources, you can use the helm uninstall command followed by the release name:

```shell
helm uninstall myapp
```

## Helm Templating

Helm provides a powerful templating engine that allows you to generate dynamic Kubernetes manifests based on the values provided during installation or upgrade. Templating enables you to customize the configuration of your application by manipulating YAML files using Go templates.

### Template Functions

Helm provides a set of built-in functions that you can use within your templates to perform various operations. These functions allow you to manipulate strings, perform arithmetic operations, iterate over lists, and more.

Here's an example of using the `tpl` function to concatenate two strings in a template:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ tpl .Release.Name "-" .Chart.Name }}
```
{: file="service.yaml" }

In the above example, the `.Release.Name` represents the release name, and `.Chart.Name` represents the chart name. The `tpl` function concatenates these two values using a hyphen(-).

### Conditional Statements

Helm templates support conditional statements that allow you to conditionally include or exclude YAML blocks based on certain conditions.

This feature is useful when you want to generate different configurations based on the values provided:

```yaml
apiVersion: v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: {{ .Values.replicaCount }}
{{- if eq .Values.environment "production" }}
  strategy:
    type: Recreate
{{- end }}
  template:
    spec:
      containers:
        - name: myapp
          image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
```
{: file="deployment.yaml" }

In the above example, the `if` statement checks if the `environment` value is set to "production." If it matches, it includes the `strategy` block in the generated manifest.

### Loops

Helm templates also support loops, allowing you to iterate over lists or maps and generate multiple YAML blocks based on the values.

This feature is helpful when you have multiple replicas or need to generate resources for each item in a list, for example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
{{- range .Values.ports }}
  - port: {{ .port }}
    targetPort: {{ .targetPort }}
{{- end }}
```
{: file="service.yaml" }

In the above example, the `range` statement iterates over the `ports` list and generates multiple `port` blocks based on the values provided in the `values.yaml` file.

### Customizing Template Values

You can customize templated values by modifying the `values.yaml` file or by providing a separate value file during installation or upgrade. This allows you to override default values and customize the generated manifests, for example:

```yaml
# values.yaml
replicaCount: 3
image:
  repository: myapp
  tag: 1.0.0
ports:
  - port: 8080
    targetPort: 80
  - port: 8443
    targetPort: 443
```
{: file="values.yaml" }

By modifying the values in the `values.yaml` file, you can control the number of replicas, the image repository and tag, and the ports exposed by your application.

## Conclusion

Helm is a powerful tool that simplifies the deployment and management of applications on Kubernetes clusters. With Helm, you can define reusable charts that encapsulate all the necessary Kubernetes resources and configurations. By using Helm charts, you can easily install, upgrade, and remove applications in a consistent and reproducible manner.

Templating enables you to customize the configuration of your application based on the values provided during installation or upgrade. With the ability to use functions, conditional statements, and loops, you have the flexibility to generate complex configurations that adapt to your specific needs.

Happy Helm-ing!
