---
layout: page
title: First Time
parent: Connecting to Sulis
grand_parent: Getting Started
nav_order: 1
---

# Connecting to Sulis for the First Time
{: .no_toc }

1. TOC
{:toc}

## Before you connect

### SSH public keys

Access to Sulis is by SSH public key and you must therefore have an SSH public key associated with your account before you login. If you did not [provide an SSH public key]({% link gettingstarted/getaccount.markdown %}#username-and-public-key-upload) when you requested a machine account on Sulis, follow the instructions at [Managing your SSH public keys]({% link gettingstarted/connecting/managingsshkeys.markdown %}) to add one.

### Time-based One-Time Passwords

We strongly recommend your review the page on [Time-based One-Time Passwords]({% link gettingstarted/connecting/TOTP.markdown %}) (TOTP) before you make your first attempt at connecting to Sulis. You should additionally be aware of the following known SSH client issues that may affect you during your first attempt to login:

* [**Putty**](https://www.chiark.greenend.org.uk/~sgtatham/putty/) users must ensure that their terminal window does not close automatically when they first login, otherwise they will be automatically registered for 2FA but will have missed the QR code generated. This behaviour can be controlled on the main "Session" window by selecting the "Never" option in the "Close window on exit" section of the window, before connecting to Sulis.
* [**MobaXterm**](https://mobaxterm.mobatek.net/) users may find their initial connection attempt resulting in the message "authenticated with partial success" but with the QR code not shown. If this occurs for you, contact your relevant support team to have your TOTP reset and use a different SSH client (such as Putty - noting the comment above) to make the next connection attempt. Once you have registered your TOTP you should then be able to use MobaXterm to connect to Sulis.
* **MacOS** users may need to temporarily reduce the line spacing in their terminal in order for the QR code to scan correctly.

### Access restrictions

Access to Sulis is restricted to a limited set of IP addresses. Please contact your [corresponding support team]({% link support.markdown %}) if you are unable to connect to Sulis from your location.

## Your first connection

The first time you connect to Sulis you will be automatically registered for a [TOTP]({% link gettingstarted/connecting/TOTP.markdown %}) and your session will immediately log out. You will be presented with a QR code and five single use emergency codes (we recommend you make a note of these codes). You will need to scan the QR code with a TOTP compatible app such as Google Authenticator or Microsoft Authenticator (see the [TOTP]({% link gettingstarted/connecting/TOTP.markdown %}#totp-clients) page for more options). An entry will be added to your app labelled as __login.sulis.ac.uk__ and with a constantly changing 6 digit code (the 2FA verification code). When you next try to login to Sulis you will be asked for this 2FA verification code.

The hostname for connecting to Sulis is: **login.sulis.ac.uk**