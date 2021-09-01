---
layout: page
title: TOTP
parent: Connecting to Sulis
grand_parent: Getting Started
nav_order: 3
---

# What is TOTP

TOTP or Time-based One-Time Passwords is an algorithm utilising the current time to generate a one time password. [TOTP](https://en.wikipedia.org/wiki/Time-based_One-Time_Password) is a common method for adding a second factor of authentication to user accounts.

# TOTP Clients

Sulis has mandatory two-factor authentication so you will need a mobile app or desktop program installed on your computer or mobile phone before you login for the [first time](./firsttime.html). Below we have listed some applications which are compatible with the TOTP algorithm although we do not endorse any particular client and please show due caution when installing any application.

## Mobile

* andOTP (Android/Play Store)
* FreeOTP (Android, IOS)
* Google Authenticator (iOS, Android/Play Store)
* Microsoft Authenticator (iOS, Android/Play Store)

## Desktop

* Authenticator (Linux/Flathub, Linux/Snap Store)
* KeePass using one of several plugins available (Windows, macOS, Linux, BSD)
* KeePassXC (Windows, macOS, Linux)
* OTPClient (Linux/Flathub)
* otp-manager (Windows, macOS)
* Pass using pass-otp plugin (macOS via Brew, Linux)
* WinAuth (Windows)

## Desktop and Mobile

* Authy (Android, IOS, MacOS, Windows, Linux)

# Resetting TOTP

If you still have login access to your account (i.e. you just want to change your TOTP or you've used an single use emergency code to login) then you can reset your TOTP by running:

```
{{site.data.terminal.prompt}} 2fa-reset
```

at the comand prompt.

If you can no longer login to your account on Sulis, you will need to contact your [local support team](../../support/) and ask for your TOTP to be reset.
