---
layout: page
title: Connecting to Sulis
parent: Getting Started
has_children: true
nav_order: 2
---

Interactive access to Sulis can be achieved via a [Secure Shell (or SSH)](https://en.wikipedia.org/wiki/Secure_Shell_Protocol) client. Data may additionally be transferred to or from Sulis using SSH.

Two-factor authentication is enabled on Sulis meaning that two forms of identication are required to authenticate your access. These two forms of identification are:

* An SSH keypair protected by a passphrase.
* A [Time-based One-Time Password (or TOTP)](./TOTP.html).

Before attempting to connect to Sulis, please make sure you read the page on connecting for the [First Time](./firsttime.html).