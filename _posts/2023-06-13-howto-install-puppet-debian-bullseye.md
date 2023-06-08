---
layout: post
title:  "HowTo Install Puppet Server on Debian Bullseye"
description: "Learn how to install the latest version of Puppet Server (and clients) on Debian Bullseye, in this step-by-step guide."
date:   2023-06-13
tags: devops puppet howto debian
---

![A programer controling multiple machines](/assets/puppet.png)

## Introduction

Puppet Server is an essential tool in the realm of IT automation. It enables system administrators to manage the configuration of their systems, ensuring consistency and reliability across their infrastructure. Keeping Puppet Server updated is crucial as each new version brings improvements, new features, and security fixes that help maintain an efficient and secure environment.

## Prerequisites

Before we dive into the installation process, there are a few prerequisites to check:

- A system running Debian Bullseye. You can check your Debian version by opening your terminal and typing `lsb_release -a`.
- Sufficient system resources. The latest version of Puppet Server requires at least 2GB of RAM and a modern processor. Check your system's resources by using the `free -m` and `lscpu` commands respectively.
- Root or sudo access. The installation process requires administrative privileges. Make sure you have these permissions by typing `sudo -v`.

## Step-by-step Guide to Install the Latest Version of Puppet Server

### Step 1: Update System Packages

First, update your system packages by running the following command:

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

### Step 2: Add Puppet Repository

Next, add the Puppet repository to your system:

```bash
wget https://apt.puppetlabs.com/puppet-release-bullseye.deb
sudo dpkg -i puppet-release-bullseye.deb
sudo apt-get update
```

### Step 3: Install Puppet Server

Now, you can install Puppet Server:

```bash
sudo apt-get install puppetserver
```

### Step 4: Configure Puppet Server

Edit the Puppet Server configuration file to specify the amount of RAM for the server:

```bash
sudo nano /etc/default/puppetserver
```

Change the line that starts with JAVA_ARGS to -Xms2g -Xmx2g to the amount of RAM you can dedicate to your puppetserver.

### Step 5: Start and Enable Puppet Server

Finally, start and enable Puppet Server:

```bash
sudo systemctl start puppetserver
sudo systemctl enable puppetserver
```

### Step 6: Installing and Using r10k (optional)

r10k is a code management tool for Puppet. It allows you to manage your environment configurations, such as production, testing, and development, in a source control repository and automatically install [modules from the forge](https://forge.puppetlabs.com/) on your Puppet Server.

To install r10k, run:

```bash
sudo /opt/puppetlabs/puppet/bin/gem install r10k
```

To use r10k, you'll need to create a [Puppetfile](https://www.puppet.com/docs/pe/2023.2/puppetfile.html) which describes the modules you want to use, and then use r10k to deploy them with the following command:

```bash
sudo r10k puppetfile install
```

### Step 7: Installing and Using eyaml (optional)

eyaml is a Puppet ENC (External Node Classifier) that allows you to store encrypted data in your Puppet code.

To install eyaml, run:

```bash
sudo /opt/puppetlabs/puppet/bin/gem install hiera-eyaml
```

To use eyaml, you'll need to create keys, encrypt your data, and then you can use it in your Puppet code.

#### Creating Keys for eyaml

Before you can start encrypting your data with eyaml, you need to create a pair of keys. Here's how you can do it:

```bash
cd /etc/puppetlabs/puppet
sudo eyaml createkeys
```

This will create a pair of keys, a public key (public_key.pkcs7.pem) and a private key (private_key.pkcs7.pem), in the /etc/puppetlabs/puppet/keys directory. The public key will be used to encrypt data, and the private key will be used to decrypt it.

#### Encrypting and Storing Sensitive Data

Let's say you have a password that you want to encrypt and use in your Puppet code. Here's how you can do it:

Encrypt the password:

```bash
eyaml encrypt -s 'mysecretpassword'
```

This will output an encrypted block that looks something like this:

```bash
ENC[PKCS7,MIIBeQYJKoZIhvcNAQcDoIIBajCCAWYCAQAxggEhMIIBHQIBADAFMAACAQEwDQYJKoZIhvcNAQEBBQAEggEALs1gZEHuetdVTxR3H9Vz3vG6RZ4VQZ4Fee2y2A7s5O6bqQj3u4q9f8W3Z4V]
```

Use the encrypted password in your Puppet code:

You can use the encrypted password in your Puppet code like this:

```puppet
$user_password = lookup('user::password')
user { 'myuser':
  ensure   => present,
  password => $user_password,
}
```

In your Hiera data, you would have something like this:

```yaml
user::password: >
  ENC[PKCS7,MIIBeQYJKoZIhvcNAQcDoIIBajCCAWYCAQAxggEhMIIBHQIBADAFMAACAQEwDQYJKoZIhvcNAQEBBQAEggEALs1gZEHuetdVTxR3H9Vz3vG6RZ4VQZ4Fee2y2A7s5O6bqQj3u4q9f8W3Z4V]
```

This way, the sensitive data is safely encrypted and can be pushed to a repository.

Remember to keep your private key safe and secure. If it's compromised, anyone can decrypt your data. Also, make sure to back it up. If you lose it, you won't be able to decrypt your data.

### Verifying the Installation

You can check if Puppet Server, r10k, and eyaml are running and their versions by running:

```bash
puppetserver --version
r10k version
eyaml version
```

## Installing Puppet Agent and Connecting to Puppet Server

Puppet Agent is the client-side tool which handles the application of configurations compiled on the Puppet Server.

To install Puppet Agent on another machine, run:

```bash
wget https://apt.puppetlabs.com/puppet-release-bullseye.deb
sudo dpkg -i puppet-release-bullseye.deb
sudo apt-get update
sudo apt-get install puppet-agent
```

To connect the Puppet Agent to the Puppet Server, follow these steps:

### Configure your PATH to access Puppet commands:

```bash
export PATH=/opt/puppetlabs/bin:$PATH
```

### Configure the server setting by editing the puppet.conf file:

```bash
sudo nano /etc/puppetlabs/puppet/puppet.conf
```

### Add the following lines:

```bash
[main]
server = <your-puppet-server-hostname>
```

Replace <your-puppet-server-hostname> with the hostname of your Puppet Server.

### Connect the agent to the primary server and sign the certificate

On the Puppet Agent machine, run:

```bash
sudo puppet agent --test
```

This will create a certificate signing request (CSR) which is sent to the Puppet Server.

On the Puppet Server

```bash
sudo puppetserver ca list
```

Sign the CSR for your Puppet Agent:

```bash
sudo puppetserver ca sign --certname <your-puppet-agent-hostname>
```

Replace <your-puppet-agent-hostname> with the hostname of your Puppet Agent machine.

Now, your Puppet Agent should be connected to the Puppet Server.

## Conclusion

In this guide, we've walked through the process of installing the latest version of Puppet Server on Debian Bullseye, along with the installation and usage of r10k and eyaml. We've also covered how to install and connect a Puppet Agent to the Puppet Server. Keeping your Puppet Server and its components updated is crucial for maintaining an efficient and secure IT environment. Happy automating!