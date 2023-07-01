---
layout: post
title: "Swift Development on the iPad using Terraform and Digital Ocean"
date: 2020-02-11
categories: swift ipad
tags: swift ipad remote-development iac infrastructure-as-code terraform digital-ocean ssh mosh
image:
    path: /assets/images/ipad-development-terraform-digital-ocean.webp
---

Since the release of iPadOS 13, the iPad has gained a lot of useful features making it so much more of a viable option for getting real work done. That being said, it still lacks in several areas, especially in the area of Swift development.

Sure, Apple does have its own Swift Playground app, but as the name suggests, it is nothing more than a playground to learn and for prototyping concepts. Although Xcode on the iPad will be a dream come true for many of us, I don’t see that happening anytime soon. 

In this post, we’ll take a look at how to set up a remote machine for Swift development on the iPad with Terraform and Digital Ocean.

## Prerequisites

* A Digital Ocean account
* The Blink shell emulator app (or similar)
* A Mac
* And of course, an iPad

## Setting up a Digital Ocean account

Digital Ocean, like AWS, offers a wide range of cloud products and services, ranging from VPS aka Droplets to Kubernetes Clusters. Head on over to their website and [create your account ](https://m.do.co/c/511f92eb87d6).

## Create your Personal Access Token

Once you have your Digital Ocean account created, you need to create your Personal Access Token which you will later need when creating your infrastructure using Terraform.

You can create a new Personal Access Token on Digital Ocean by navigating to Manage > API and then clicking on the button to the right “Generate New Token”. Note that you need to enable both the READ and WRITE scope when creating your token.

**Make sure you copy and save your token somewhere safe for easy access later.**

## Generating and adding your SSH Keys

In order to access our machine via SSH we need to generate a new set of SSH key pairs for both our Mac and iPad.

### On your Mac

On macOS this can easily be done by running the following command:

`ssh-keygen -o`

This will generate a new public and private key pair and our **public key** is what we will need to add on Digital Ocean. Copy the contents of `~/.ssh/id_rsa.pub` or copy it to the clipboard with: `pbcopy < ~/.ssh/id_rsa.pub`.

Next, we need to add our public key on Digital Ocean under the Account > Security > SSH keys section. Click on the “Add SSH Key” button and paste in your public key. 

Now would also be a good time to take note of the Fingerprint generated for your SSH key which you’ll need later. 

### On your iPad

On the iPad, there are a number of different apps that you can use to generate a new SSH key pair including [Prompt](https://panic.com/prompt/), [Termius](https://termius.com), [Working Copy](https://workingcopyapp.com) and [Blink shell](https://www.blink.sh) (my personal favorite).

Using Blink shell you can either generate a new SSH key pair by simply typing `ssh-keygen` from the prompt or by going to Settings > Keys and adding it there. Tap on the keys you just added and select “Copy Public Key”.

Repeat the steps for adding the public key for your Mac on Digital Ocean and remember to also take note of the generated fingerprint.

## Creating your VPS using Terraform

In case you are not yet familiar with [Terraform](https://www.terraform.io), Terraform uses Infrastructure as Code to provision and manage cloud infrastructure and services. It automates rather tedious tasks when creating and setting up cloud infrastructure. 

Of course, we can also set up everything manually by hand but using Terraform to automate the entire process offers a lot of benefits as you will see in a minute.

### Installing Terraform

Terraform offers a binary package for just about every platform, for more information see the [installation guide](https://learn.hashicorp.com/terraform/getting-started/install.html).

On macOS I prefer installing most of my tools using [Homebrew](https://brew.sh) which is as simple as running the following command:

```shell
brew install terraform
```

## Infrastructure code

Let’s create a new terraform script called `dev-machine.tf`:

```yaml
# Variable declaration
variable "do_token" {}

variable "pub_key" {}

variable "pvt_key" {}

variable "ssh_fingerprint" {}

# Configure the DigitalOcean Provider
provider "digitalocean" {
  token = "${var.do_token}"
}

# Provider setup
resource "digitalocean_droplet" "web" {
  image  = "ubuntu-18-04-x64"
  name   = "ubuntu-dev"
  region = "fra1"
  size   = "s-1vcpu-1gb"
  private_networking = true
  ssh_keys = [
    var.ssh_fingerprint,
  ]
  # Connection setup
  connection {
    user        = "root"
    type        = "ssh"
    host        = self.ipv4_address
    private_key = file(var.pvt_key)
    timeout     = "2m"
  }
  # Provisioning
  provisioner "file" {
    source      = "provision.sh"
    destination = "/tmp/provision.sh"
  }
  provisioner "remote-exec" {
    inline = [
      "chmod +x /tmp/provision.sh",
      "/tmp/provision.sh",
    ]
  }
}
```
{: file="dev-machine.tf" }

Let’s break down each section of the terraform script above:

### Variable declaration

We declare variables at the beginning of the file that we will reference later throughout the script. Here we create variables for our Digital Ocean Personal Access Token, public key, private key, and SSH fingerprints.

### Provider setup

The provider section tells Terraform that we’re going to use Digital Ocean as our provider. Terraform also supports other providers like AWS, Azure, Linode, and [many more](https://www.terraform.io/docs/providers/index.html). Here we create a very small VPS (droplet) with 1 CPU, 1GB of RAM, 25GB of SSD storage as defined by `size = "s-1vcpu-1gb"`. Remember to also set your region accordingly, in the example it is set to Frankfort, Germany `region = "fra1"`.

### Connection setup

In the connection section, we define how Terraform will access our newly created machine when running the provision script over SSH.

### Provisioning

Once we have our newly created machine up and running we can start automating the process of installing the necessary tools and packages by running the provisioning script.

Create a new bash script file called `provision.sh` in the same directory as `dev-machine.tf`:

```bash
#!/bin/bash

# update
apt -y update && apt -y install unattended-upgrades

# dependencies
apt -y install libcurl3 libpython2.7 libpython2.7-dev

# tools
apt -y install curl
apt -y install mosh
apt -y install clang

# swift
wget https://swift.org/builds/swift-5.0.1-release/ubuntu1804/swift-5.0.1-RELEASE/swift-5.0.1-RELEASE-ubuntu18.04.tar.gz
tar xzf swift-5.0.1-RELEASE-ubuntu18.04.tar.gz
mv swift-5.0.1-RELEASE-ubuntu18.04 /usr/share/swift
echo "export PATH=/usr/share/swift/usr/bin:$PATH" >> ~/.zshrc
rm swift-5.0.1-RELEASE-ubuntu18.04.tar.gz

# vapor
eval "$(curl -sL https://apt.vapor.sh)"
apt -y install vapor

# optional tools
apt -y install zsh
chsh -s $(which zsh)
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended
apt -y install tmux

# firewall rules
sudo ufw allow 60000:61000/udp
```
{: file="provision.sh" }

Let’s take a minute to examine what exactly the script does:

### Updating system packages

First things first, we do a simple `apt update` and `apt upgrade` to get our system up to date.

### Installing dependencies

Next, we install all the dependencies needed for the tools and packages that we’re going to install.

### Installing tools

We need to make sure that we also install all the tools that we’re going to need.

### Swift & Vapor

Now that we have all the required packages and tools installed we can finally proceed with installing both Swift and Vapor.

### Optional tools

This part of the script is entirely up to you and of course optional. I do however suggest that you install them and familiarise yourself with them, especially [Tmux](https://github.com/tmux/tmux/wiki) which will make your development experience on the iPad so much more enjoyable. 

Since we won’t be able to run an IDE, the next best option would be to customize and improve the experience while working in a shell environment. To achieve this I also recommend installing the [ZSH](https://www.zsh.org) shell together with [OHMyZSH](https://ohmyz.sh).

### Firewall rules

Finally, we need to open the inbound and outbound UDP ports on port `60000` and `61000` which is needed by Mosh. [Mosh](https://mosh.org) is a remote terminal application that allows roaming, supports intermittent connectivity and provides intelligent local echo and line editing of user keystrokes. It is a great alternative to SSH while working on the iPad when a stable connection is not always guaranteed.

## Creating the infrastructure

On your Mac, run the following two commands in terminal:

```shell
export DO_TOKEN=YOUR-PERSONAL-ACCESS-TOKEN-HERE && export SSH_FINGERPRINT=YOUR-MAC-SSH-FINGERPRINT
```

Next we need to initialise our Terraform script:

```shell
    terraform init
```

Before you go ahead and run the Terraform script, it is always a good idea to first run `terraform plan`. This will print out a diff of what Terraform is about to do without actually doing it, allowing you to make sure everything is correctly configured.

Finally, and where all the magic happens, we can apply our changes by running:

```shell
terraform apply \
-var "do_token=$DO_TOKEN" \
-var "pub_key=$HOME/.ssh/id_rsa.pub" \
-var "pvt_key=$HOME/.ssh/id_rsa" \
-var "ssh_fingerprint_mac=YOUR-MAC-SSH-FINGERPRINT" \
-var "ssh_fingerprint_ipad=YOUR-IPAD-SSH-FINGERPRINT"
```

## Wrapping up

That's it, once completed, Terraform will create the infrastructure, install all the tools, and correctly configure everything for us to connect to our remote development machine.

On Digital Ocean you should now see your newly created Droplet with a publicly accessible IP address and hostname.

You can now go ahead and connect from your iPad using either SSH or Mosh. Simply add a new host using either the public IP address or hostname and connect via SSH or Mosh. Using Blink shell you can connect with Mosh running `mosh your-server-name`.
