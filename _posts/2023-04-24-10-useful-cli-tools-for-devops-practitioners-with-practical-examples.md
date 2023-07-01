---
layout: post
title: "10 Useful CLI tools for DevOps Practitioners with practical examples"
date: 2023-04-24
categories: devops tools
tags: devops cli-tools command-line terminal homebrew aws kubernets http json fuzzy-search github
image:
    path: /assets/images/useful-cli-tools-for-devops-practitioners.png
---

As a DevOps practitioner, you're probably always on the lookout for ways to improve your workflow and be more productive.

One way to do this is by using command-line tools or CLI tools. CLI tools are programs that run in the terminal or command prompt, and they can help you automate repetitive tasks, manage your infrastructure, and more.

In this post, we'll explore 10 of my favorite CLI tools:

- [Homebrew](#homebrew)
- [AWS CLI](#aws-cli)
- [k9s](#k9s)
- [stern](#stern)
- [kubectx & kubens](#kubectx--kubens)
- [xh](#xh)
- [jq](#jq)
- [jqp](#jqp)
- [fzf](#fzf)
- [gh](#gh)

## [Homebrew](#homebrew)

[Homebrew](https://brew.sh), or `brew` for short, is a package manager for macOS that allows you to install and manage CLI tools and other software packages. It has a simple and user-friendly CLI that allows you to search for, install, and update packages with just a few commands. 

Using brew, you can install and manage CLI tools and other dependencies, including those mentioned in this post.

**Usage**

```shell
brew install <package-name>
or
brew install --cask <application-name>
```

> Remember to regularly do a `brew update` and `brew upgrade` to ensure that you are using the latest versions of your install packages/tools.
{: .prompt-tip }


## [AWS CLI](#aws-cli)

If you're using Amazon Web Services (AWS), the [aws CLI](https://aws.amazon.com/cli/) is a must-have tool. It allows you to manage your AWS resources using a CLI, including EC2 instances, S3 buckets, and more. 

The AWS CLI is an essential tool if you need to manage AWS resources programmatically.

**Usage**

Copy a S3 resource to a local path:

```shell
aws s3 cp s3://<resource-name> <local-path>
```

## [k9s](#k9s)

[k9s](https://github.com/derailed/k9s) is a CLI tool for Kubernetes that provides a terminal-based UI for managing and monitoring your Kubernetes clusters. It allows you to view resources, logs, and metrics for your clusters, and it has a built-in search and filtering capabilities. 

With k9s, you can easily manage your Kubernetes clusters without leaving the terminal.

**Usage**

Run k9s for a specific namespace:

```shell
k9s -n <namespace>
```

## [stern](#stern)

[stern](https://github.com/stern/stern) is another powerful CLI tool for Kubernetes that allows you to tail logs from multiple pods at once. It has a simple and intuitive CLI that allows you to quickly switch between pods and namespaces, and it supports filtering and highlighting based on keywords. 

stern is a great tool for debugging and monitoring your Kubernetes applications, and it can save you a lot of time compared to manually searching through logs.

**Usage**

Show logs for specific pods:

```shell
stern <regex-or-pod-name>
```

## [kubectx & kubens](#kubectx--kubens)

[kubectx](https://github.com/ahmetb/kubectx) is a CLI tool for managing and switching between Kubernetes contexts. It allows you to easily switch between different Kubernetes clusters with a simple command, without having to remember long and complex commands.

Simply run the `kubectx` command and select a context from the list.

[kubens](https://github.com/ahmetb/kubectx) is another useful CLI tool for Kubernetes that works alongside kubectx. With kubens, you can quickly switch between different namespaces, and avoid mistakes caused by accidentally running commands on the wrong namespace. 

## [xh](#xh)

[xh](https://github.com/ducaale/xh) (formerly known as httpie) is a CLI tool for making HTTP requests that provides a more user-friendly and intuitive interface than traditional command-line tools such as curl. 

It supports a variety of HTTP features such as JSON and form data, file uploads, authentication, and more, and it has a syntax that's easy to read and understand. xh is a great tool if you need to interact with web APIs and test HTTP endpoints. 

With xh, you can quickly and easily make HTTP requests from the command line, without having to remember complex syntax or multiple options.

**Usage**

Query a web API endpoint using request parameters:

```shell
xh httpbin.org/json id==5
```

Output:

```shell
HTTP/1.1 200 OK
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Length: 429
Content-Type: application/json
Date: Mon, 24 Apr 2023 18:22:26 GMT
Server: gunicorn/19.9.0

{
    "slideshow": {
        "author": "Yours Truly",
        "date": "date of publication",
        "slides": [
            {
                "title": "Wake up to WonderWidgets!",
                "type": "all"
            },
            {
                "items": [
                    "Why <em>WonderWidgets</em> are great",
                    "Who <em>buys</em> WonderWidgets"
                ],
                "title": "Overview",
                "type": "all"
            }
        ],
        "title": "Sample Slide Show"
    }
}
```

## [jq](#jq)

[jq](https://github.com/jqlang/jq) is a lightweight and powerful command-line tool for parsing JSON data. It allows you to filter, transform, and format JSON data using a simple syntax. jq is an essential tool if you work with JSON data regularly.

**Usage**

Filter `aws s3api` JSON response to determine whether accelerated transfer is enabled on a specific bucket:

```shell
aws s3api get-bucket-accelerate-configuration --bucket <bucket-name> | jq '.Status' -r || true
```

Output:

```shell
Enabled
```

## [jqp](#jqp)

[jqp](https://github.com/noahgorstein/jqp) is a command-line tool for parsing and querying JSON data. It allows you to filter and transform JSON data using a SQL-like query language. 

jqp is similar to the previously mentioned jq tool, but it has some additional features, such as support for joins, aggregation, and subqueries. jqp is a great tool if you work with complex JSON data and need a powerful and flexible way to extract and manipulate data. 

With jqp, you can quickly extract and transform data from JSON files or APIs, and integrate it with your other CLI tools and workflows.

**Usage**

Query a web api enpoint and pipe the JSON response with jqp:

```shell
xh httpbin.org/json id==5 | jqp
```

From jqp you can then query the JSON response, for example:

```shell
.slideshow.slides[0].title
```

Output:

```shell
"Wake up to WonderWidgets!"
```

## [fzf](#fzf)

 [fzf](https://github.com/junegunn/fzf) is a command-line fuzzy finder tool that allows you to quickly search for files, directories, and other items in your file system. It provides a fast and interactive way to search and navigate your file system, with support for keyboard shortcuts and customizable search options.

 fzf is a great tool when you work with large and complex file systems and want a more efficient way to navigate and search them.

 With fzf, you can quickly search for files, directories, and other items in your file system, and open them in your preferred editor or tool with just a few keystrokes.

**Usage**

Checkout out a recent branch using Git and fzf:

```shell
git branch --sort=-committerdate | fzf --header "Checkout Recent Branch" --preview "git diff --color=always {1}" | xargs git checkout
```

Running the above command will present a list of recent Git branches that you can quickly check out.

> Pro Tip: Create an alias like `gbr` using the above command to avoid having to type out the full command every time.

## [gh](#gh)

 [gh](https://github.com/cli/cli) is a CLI tool developed by GitHub that allows you to interact with GitHub from the command line. It provides a convenient way to create, view, and manage issues and pull requests, as well as perform other common GitHub tasks such as cloning repositories, creating releases, and managing workflows.

 With gh, you can quickly perform common tasks without having to switch between the command line and the GitHub web interface.

**Usage**

Create a Pull Request on GitHub with main as the base branch:

```shell
gh pr create -B main
```

Running the above command will open the GitHub web interface in your browser and start creating the Pull Request.
