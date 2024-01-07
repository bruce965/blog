---
title: Generate RSA Key Pairs on Windows or Linux
date: 2017-02-07 08:47:54 +0100
categories: [Software, Git]
tags: [Git, Windows, Linux]
img_path: /assets/img/posts/2017/
---

This is a quick guide to generate key pairs on Windows or Linux. Includes an (optional) introduction to asymmetric cryptography. If you don't have time to waste, feel free to jump to [Windows TL;DR](#windows-tldr) or [Linux TL;DR](#linux-tldr).

## Intro to Asymmetric Cryptography

In a **symmetric cryptography** system, there is usually just **one key to either encrypt or decrypt**. If you know the key you can both read and write encrypted messages.

An **asymmetric cryptography** system, on the other hand, works with **two keys**: the public key and the private key. The **public key can only encrypt messages**, while the **private key can only decrypt**. A message cannot be decrypted without the public key, and it cannot be encrypted without the private key.

Asymmetric cryptography can also be used to create **digital signatures**. When used for signing, the use of the keys is inverted: messages are **signed with the private key**; anyone can later **validate the signature with the public key**.

A message cannot be signed without the private key. This idea can be used to authenticate you: you are supposed to be the only possessor of your private key; anyone with your public key can challenge you to prove your identity; if you have the key, you can stand the challenge.

So you must **never share your private key** with anyone, keep it safe somewhere. But, depending on the scenario, you might either want to **do share your public key** with the whole world, or just with someone you want to able to decode your messages or authenticate you.

> Note: you can read more on the subject at [Asymmetric cryptography](https://en.wikipedia.org/wiki/Asymmetric_cryptography) Wikipedia page.

## Linux

And so, let's generate an RSA key pair now.

The most common public/private key format is **OpenSSH format**, this is the format Linux users will be generating.

If you are on Linux, you need `openssl` package to be installed on your system. This package is already included in most distributions.

Fire up your terminal and generate a new **2048 bit RSA** private key with the following command.

```bash
openssl genpkey -out mykey.pem -algorithm rsa -pkeyopt rsa_keygen_bits:2048
```

> Note: you can check [`man openssl`](http://linuxcommand.org/man_pages/openssl1.html) and tune the parameters if you need something different.

You will probably also need your public key. You can derive it from the private key by running the following command.

```bash
openssl rsa -in mykey.pem -out mykey.pub -pubout
```

## Windows

Windows users also have another option: PuTTY format. You should only store your key in this format if you are working with PuTTY, for any other use please go with OpenSSH format.

Look for *puttygen.exe* on [PuTTY website](http://www.putty.org/).

![PuTTY Key Generator main screen.](rsa-puttygen.png)

With the default options, *PuTTY Key Generator* will generate a **2048 bit RSA** key, this is probably what you need. You can change the key type if you want something different.

Click the *"Generate"* button and start moving the mouse around to generate some random bit. This bits are required so that the key can really be unique to you, based on how you moved the mouse.

Once your key pair is ready, you can either export your private key in PuTTY format by clicking *"Save private key"*, or in **OpenSSH format** by clicking *"Conversions" » "Export OpenSSH key"*.

The public key in OpenSSH format lies in the upper text field.

> Note: you don't need to save your RSA public key if you don't need it now. You can always extract if from your RSA private key later.

---

## Linux TL;DR

```bash
# Generate a 2048 bit RSA private key
openssl genpkey -out mykey.pem -algorithm rsa -pkeyopt rsa_keygen_bits:2048

# Extract the public key
openssl rsa -in mykey.pem -out mykey.pub -pubout
```

---

## Windows TL;DR

1. Download [puttygen.exe](https://the.earth.li/~sgtatham/putty/latest/x86/puttygen.exe);
2. Click *"Generate"* and move the mouse around a bit.

If you need your key in PuTTY format, click *"Save private key"*; or if you need your key in OpenSSH format, click *"Conversions" » "Export OpenSSH key"*.

You can also copy your public key in OpenSSH format from the upper text field.

> Note: if the link is broken, you can look for *puttygen.exe* on [PuTTY website](http://www.putty.org/).
