---
layout: post
title: "Getting started with Vapor"
date: 2019-02-08
categories: swift vapor
tags: vapor server-side swift
image:
    path: /assets/images/vapor-swift.webp
---

Getting started with [Vapor](https://vapor.codes) is easy thanks to the handy Vapor Toolbox and well written [documentation](https://docs.vapor.codes/3.0/).

## Prerequisites

* Swift 4.1 (bundled with Xcode)
* Xcode 9.3 or higher (optional if you install Swift manually)

## Installing Vapor

The preferred method of installation is via [brew](https://brew.sh/).

To install Vapor run to following commands in Terminal:

```shell
brew tap vapor/tap && brew install vapor/tap/vapor
```

## Creating a new Vapor Project

With Vapor now installed, you can now create a new project:

```shell
vapor new api-gateway
```

This will use the default `--api` template but you can also use the `--web` template for example:

```shell
vapor new web-ui --web
```

### Building your project

Next you will need to build your project:

```shell
vapor build
```

The initial build will take some time as it needs to download and compile all the dependencies. Subsequent builds will be much quicker.

### Running your project

Now that your project is built, you can simply run it with the following command:

```shell
vapor run
```

With your server running you can expect to see the following output:

```shell
Running default command: .build/debug/Run serve
Server starting on http://localhost:8080
```

### Working with Xcode

Since Vapor leans on Swift Package Manager you don't need Xcode to get started with Vapor. You could use any Text Editor of your choice especially if you're not running macOS. If you are however running macOS you would most probably want to use Xcode by creating an Xcode project:

```shell
vapor xcode
```

This will generate an Xcode project for you allowing you to run and debug your application just like you would do for any other Xcode project.

That's it! You are now ready to write Server-Side applications using Swift.

