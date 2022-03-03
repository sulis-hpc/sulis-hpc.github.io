---
layout: page
title: Creating an account
parent: Getting Started
nav_order: 1
---

# Getting an Account
{: .no_toc }

1. TOC
{:toc}

## Eligibility

Sulis is available to users within the HPC Midlands+ consortium and to users who have been granted allocations of time by the EPSRC. It is also available to users attending training courses that use Sulis for exercises. 

Users needing exploratory access to Sulis in advance of an application via EPSRC mechanisms should contact [sulis@warwick.ac.uk](mailto:sulis@warwick.ac.uk). 

## Accessing SAFE

All Sulis users must register with SAFE, an account management and administration portal for HPC services run by the [Edinburgh Parallel Computer Centre (EPCC)](https://www.epcc.ed.ac.uk/). 

SAFE can be accessed at [https://safe.epcc.ed.ac.uk/](https://safe.epcc.ed.ac.uk/).

If you do not already have an account with SAFE then you will need to create one. We recommend using the option to login with your institutional credentials to benefit from the account security
management provided by your home institution. 

If you are unfamiliar with SAFE, consider reading the [SAFE website guide](https://epcced.github.io/safe-docs/).

## Joining a Sulis project

Having signed into SAFE you will be presented with a screen like the following.

![SAFE Homepage](/assets/images/01_SAFE_home_screen.png "SAFE HomePage")

To join a project (i.e. a group to which CPU or GPU resource can be allocated) select "Request access" from the "Projects" menu at the top of the homepage. 

Your project code will being with the characters "su" and will have been given to you when your application for use of Sulis was granted. 

- Users within the HPC Midlands+ consortium will be given a project code by their local research computing support team.

- Users given access via EPSRC national mechanisms will be given a project code by the Sulis support team at Warwick at the start of the corresponding allocation period. 

![Select project](/assets/images/03_Select_Project.png "Select project")

You should select "Request machine account" as the access route and then click "Next" to proceed to the next step.

## Username and public key upload

To connect to Sulis itself, you will need a login account. We do not allow access to Sulis via username/password. Instead you must generate an SSH key pair and upload the PUBLIC key to SAFE 
for use on the Sulis service. 

Your local research computing support team will be able to provide guidance on generating an SSH key pair, but please note the following.

- Your private key *must* be encrypted with a strong password.
- Do not use the same key pair for Sulis that you use for any other services. 

Use this form to specify your preferred username on Sulis, and to upload your public key. Note that on Linux/Mac systems the default location for newly generated keys is in the hidden folder `.ssh` located at the top level of your home directory, and so you may need to enable "show hidden files" or equivalent when browsing for the public key file.

![Username and key](/assets/images/05_Initial_user_and_key.png "Username and key")

Once you have uploaded the key, hit "Request". The appropriate project manager  will then need to approve this request. This will either be the person within your institution who manages access to Sulis (HPC Midlands+ users) or the designated project manager for your EPSRC project.

Once your account has been created, additional SSH public keys may be added or deleted from your account via [SAFE]({% link gettingstarted/connecting/managingsshkeys.markdown %}).

## Account generation

Once your request has been approved and the account has been created you will receive an email from "SAFE Administration". This process is not instantaneous as it requires manual approval. In some cases we may need additional information to confirm your identity.

Check your junk/spam folder if the account approval email does not arrive within two working days.

Once your account is created, refer to the section on [Connecting to Sulis](connecting) for information on first login and setting up two-factor authentication.