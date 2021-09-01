---
layout: page
title: First Time
parent: Connecting to Sulis
grand_parent: Getting Started
nav_order: 1
---

The first time you connect to Sulis you will be automatically registered for a [TOTP](./TOTP.html) and your session will immediately log out. You will be presented with a QR code and five single use emergency codes (we recommend you make a note of these codes). You will need to scan the QR code with a TOTP compatible app such as Google Authenticator or Microsoft Authenticator (see the [TOTP](./TOTP.html#totp-clients) page for more options). An entry will be added to your app labelled as __login.sulis.ac.uk__ and with a constantly changing 6 digit code (the 2FA verification code). When you next try to login to Sulis you will be asked for this 2FA verification code.

* [**Putty**](https://www.chiark.greenend.org.uk/~sgtatham/putty/) users must ensure that their terminal window does not automatically when they first login, otherwise they will be automatically registered for 2FA but will have missed the QR code generated. This behaviour can be controlled on the main "Session" window by selecting the "Never" option in the "Close window on exit" section of the window, before connecting to Sulis.
* **MacOS** users may need to temporarily reduce the line spacing in their terminal in order for the QR code to scan correctly.

The hostname for connecting to Sulis is: **login.sulis.ac.uk**