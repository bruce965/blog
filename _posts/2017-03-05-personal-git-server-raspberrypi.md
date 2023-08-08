---
title: Personal Git Server on the Raspberry Pi
date: 2017-03-05 18:17:00 +0100
categories: [Software]
tags: [Raspberry Pi, Git, Linux]
---

Do you want a Git repository to share your work among your computers but you don't want to rely on third-party services? You are lucky, it's super-easy, all you need is a Raspberry Pi or a linux computer.

> Tip: want to get started with Git? Check out my [introduction to Git on Windows]({% post_url 2017-02-01-git-windows-introduction %}), it's super-easy.

## Setup

Connect your Raspberry Pi to your network and find it's address. I will use *raspberrypi* in this tutorial, as it's the default hostname in Raspbian.

If you don't have a Raspberry Pi you can follow this tutorial on any Debian or Ubuntu computer, all the steps match exactly.

> Note: you don't really need to setup your own server to use Git, there's plenty of free Git hosts. Try with [GitHub](https://github.com/) (currently the most popular, in 2020), [BitBucket](https://bitbucket.org) or [GitLab](https://gitlab.com/) (currently my favourite).

You must create the *git* user and add public keys from your development machines.

```bash
# Acquire superuser privileges
sudo su

# Create the "git" user
adduser git
```

Create your authorized keys list with `nano /home/git/.ssh/authorized_keys`.

It should look something like the following in the end: one key per-line. You can also share a single key between all your development machines, even though this is discouraged.

```text
ssh-rsa AAAAB3NzaCAAAADAQAAABA/Long+Sequence+HereAAR6bmLNSGiWK9B git@raspberrypi
ssh-rsa AAAAB3c2uTF4V1ERAAAAR6bmLNSGi/Another+Key+HereAAAGiWKyc2E9B git@raspberrypi
```

> Tip: if you need help creating a key pair, check out my guide to [generate your own RSA key pair]({% post_url 2017-02-07-generate-rsa-key %}). Use your private key to authenticate when connecting from the development machine and copy your public key in the file above.

## Creating a New Repository

Every time you want to create a new repository you can run the following commands from the server's console:

```bash
# Impersonate the "git" user
sudo su git

# Create a new repository
git init --bare ~/myrepository.git
```

From your development machine, you can clone your repository and use it as usual.

```bash
git clone git@raspberrypi:myrepository.git
```

Feel free to leave a comment below if you have any question. And thanks for reading!
