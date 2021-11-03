---
layout: page
title: Managing your SSH Public Keys
parent: Connecting to Sulis
grand_parent: Getting Started
nav_order: 2
---

# Managing your SSH Public Keys
{: .no_toc }

1. TOC
{:toc}

You can add or delete SSH public keys from your account on [SAFE](https://safe.epcc.ed.ac.uk/).

## Adding an SSH public key

Login to [SAFE](https://safe.epcc.ed.ac.uk/). Then:

1. Go to the Menu Login accounts and select the Sulis account you want to add the SSH public key to
1. On the subsequent Login account details page click the Add Credential button
1. Select SSH public key as the Credential Type and click Next
1. Either copy and paste the public part of your SSH key into the SSH Public key box or use the button to select the public key file on your computer.
1. Click Add to associate the public SSH key part with your account

Once you have done this, your SSH key will appear on your Login account details page. It will normally take up to 30 minutes for the SSH public key to be added to your account on Sulis, at which point the key status will change to "Active". Please contact your [corresponding support team]({% link support.markdown %}) if it takes longer than 24 hours for a key to be added to your account on Sulis.

## Deleting SSH public key

Login to [SAFE](https://safe.epcc.ed.ac.uk/). Then:

1. Go to the Menu Login accounts and select the Sulis account you want to remove the SSH public key from
1. On the subsequent Login account details page click the Delete Credential button
1. Select the SSH public key you want to delete and click Delete

A request will be sent to Sulis to remove the corresponding SSH key.