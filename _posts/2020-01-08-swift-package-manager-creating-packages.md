---
layout: post
title: "Swift Package Manager - Creating Packages"
date: 2020-01-08
categories: swift
tags: swift spm swift-package-manager packages
image:
    path: /assets/images/creating-swift-packages.webp
---

Swift Package Manager, or SPM, is a tool for managing the distribution of Swift code. It is also a great tool to manage your project dependencies, allowing you to import third-party frameworks or those you developed yourself.

Now that SPM is a first-class citizen as from Xcode 11 and we are starting to see some adoption, I think it is time to have a deeper look from both a consumer and a library author perspective.

In this post, we’ll have a look at how to get started using SPM creating a basic package that we can share with others or use as a dependency in your own projects.

## What is a package?

A Swift Package is essentially a collection of source files and assets compiled and package up into a single [Module]( https://docs.swift.org/swift-book/LanguageGuide/AccessControl.html ), which can, in turn, be added as a dependency in another project. Your project may contain one or multiple packages.

## Creating a Swift package

Let’s assume that we want to create an encryption/decryption library that we’d like to share with others. The first step when creating a new package is to create a new folder with the name of the package that you’d like to create, for example:

``` shell
mkdir CipherKit && cd CipherKit
```

The next step is to generate the actual package:

```shell
swift package init
```

> Remember to name the folder accordingly as SPM will use the name of the folder as the actual package name when you don’t specify the name of the package manually. To manually specify the name of the package, use the `--name` parameter.
{: .prompt-tip }

### Types of packages

By default, SPM will initialize a static library package type when running the default initialize command.

You can manually specify the type of package you’d like to create using the `--type` parameter. As of the time of writing this post the following types of packages are supported:

- empty
- library
- executable
- system-module
- manifest

For example, we can initialize a new executable package using the following command:

```shell
swift package init --type executable
```

You can run the following command to get a list of  available options when initializing a new package:

```shell
swift package init --help
```shell
Output:

```shell
OVERVIEW: Initialize a new package

OPTIONS:
  --name   Provide custom package name
  --type   empty|library|executable|system-module|manifest
```

### The manifest file

Running the initialize command will create the initial structure of our package, including the manifest file contained in `Package.swift`. The manifest files contain important information about the package’s metadata, targets, products, and external dependencies.

Have a look at the contents of `Package.swift` which should look something like this:

```swift
// swift-tools-version:5.1
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "CipherKit",
    products: [
        // Products define the executables and libraries produced by a package, and make them visible to other packages.
        .library(
        name: "CipherKit",
        targets: ["CipherKit"]),
    ],
    dependencies: [
        // Dependencies declare other packages that this package depends on.
        // .package(url: /* package url */, from: "1.0.0"),
    ],
    targets: [
        // Targets are the basic building blocks of a package. A target can define a module or a test suite.
        // Targets can depend on other targets in this package, and on products in packages which this package depends on.
        .target(
        name: "CipherKit",
        dependencies: []),
        .testTarget(
        name: "CipherKitTests",
        dependencies: ["CipherKit"]),
    ]
)
```swift
{: file="Package.swift" }

## Adding external dependencies

Next, let’s add a remote dependency to our `CipherKit` library using another open-source crypto library called [CryptoSwift]( https://github.com/krzyzanowskim/CryptoSwift.git ), hosted on GitHub:

```swift
import PackageDescription

let package = Package(
    name: "CipherKit",
    products: [
        // Products define the executables and libraries produced by a package, and make them visible to other packages.
        .library(
        name: "CipherKit",
        targets: ["CipherKit"]),
    ],
    dependencies: [
        // Dependencies declare other packages that this package depends on.
        // .package(url: /* package url */, from: "1.0.0"),
        .package (url: "https://github.com/krzyzanowskim/CryptoSwift.git", from: "1.3.0")
    ],
    targets: [
        // Targets are the basic building blocks of a package. A target can define a module or a test suite.
        // Targets can depend on other targets in this package, and on products in packages which this package depends on.
        .target(
        name: "CipherKit",
        dependencies: []),
        .testTarget(
        name: "CipherKitTests",
        dependencies: ["CipherKit"]),
    ]
)
```shell
{: file="Package.swift" }

Alternatively, you can also use a local path to refer to a package rather than a published remote version. Simply replace `url` with `path` using the relative path to the package’s folder:

```swift
 .package (url: "../MyPackageFolder")
```

Finally you can also refer to a specific branch:

```swift
.package (url: "https://github.com/krzyzanowskim/CryptoSwift.git", .branch("master"))
```

or an exact commit:

```swift
.package (url: "https://github.com/krzyzanowskim/CryptoSwift.git", .revision("a44caef0550c346e0ab9172f7c9a3852c1833599"))
```

## A note on dependency versioning

In the example above where we add a remote dependency to another framework, we specify a version of `1.3.0`. Using [Semantic Versioning]( https://semver.org ), SPM will automatically resolve the most recent version between 1.3.0 and 2.0.0 (the next major version number).

You may however want to lock into a specific version, perhaps due to a regression introduced in a later version. In cases like this, you can specify the `.exact` version of the package:

```swift
.package (url: "https://github.com/krzyzanowskim/CryptoSwift.git", .exact "1.3.0")
```

## Building your package

Now that we have our dependencies defined, we need to build our package.

To build your package run:

```shell
  swift build
```

The initial build might take a few minutes to complete depending on the number of external dependencies you have. SPM will pull all the dependencies and then build and link those to your library. Your package is now ready for distribution and consumption.

## Working in Xcode

Up until this point, we have only been working in terminal to create and build our package. For very basic libraries we could perhaps manage using just an editor like vim or vscode but we’d still lack the more advanced functionally that Xcode offers, for example, code completion and debugging support.

Fortunately, we can also build and debug our library using Xcode. To do so we need to generate an Xcode project:

```shell
  swift package generate-xcodeproj
```

This will generate a new `.xcodeproj` file that we open with Xcode just like and other project with full code completion and debugging support.

> Remember to regenerate your Xcode project every time you add or edit your dependencies.
{: .prompt-tip }

It is also important to note that your `Package.swift` remains the source of truth *not* the Xcode project file.

## Advantages

One of the main advantages of using SPM over CocoaPods or Carthage for example is the built-in support Xcode provides. Yes, that’s right, as from Xcode 11 SPM is now a first-class citizen with full support for creating and managing dependencies.

Another advantage is that you don’t have to install any additional tools or deal with compatibility issues between different versions of Xcode. SPM is supported out of the box meaning that we have one less tool to install and keep up-to-date.

## Disadvantages

SPM still being relatively new and while only recently saw major adoption still lacks a few key features. One of them being support for binary dependencies. [SE-0272]( https://github.com/apple/swift-evolution/blob/master/proposals/0272-swiftpm-binary-dependencies.md) is an open proposal which is currently in “Accepted with revisions” state and will hopefully be implemented in the next version of Swift.

> UPDATE: Distributing Binary Frameworks are now supported in Swift 5.3.
{: .prompt-info }

## Resources

- [Swift Packages]( https://developer.apple.com/documentation/swift_packages) - Official Apple documentation for Swift Packages
- [swiftpm.co](https://swiftpm.co) - A place to find Swift packages
