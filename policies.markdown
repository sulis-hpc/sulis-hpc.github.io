---
layout: page
title: Policies
permalink: /policies/
nav_order: 8
---

# Policies

## Conditions of Use

The following conditions of use apply to the Sulis Tier 2 HPC platform. Having an account on our systems will be taken to imply acceptance of these conditions.

Users shall:

1. abide by all [University of Warwick](http://www2.warwick.ac.uk/services/gov/calendar/section2/regulations/computing/) regulations concerning use of computing facilities;
1. abide by the [JANET Acceptable Use Policy](https://community.jisc.ac.uk/library/acceptable-use-policy);
1. abide by the [Acceptable Use Policy](#acceptable-use-policy) for Sulis;
1. seek permission from their PI or Project Manager before consuming any significant quantity of computing or data storage resource, or permitting students/researchers under their supervision to do so;
1. abide by the restrictions placed on the types of computation suitable for each system, as described in these pages;
1. take all reasonable measures to ensure the security of Sulis, as directed from time to time by Sulis support staff;
1. keep abreast of information provided the SAFE-Sulis mailing list and these pages about maintenance, downtime and changes to Sulis;
1. absolve the Sulis support staff from any liability arising from loss or inaccessibility of data held on our systems.
1. acknowledge use of Sulis in all publications and other scientific outputs resulting wholly or in part from use of the platform.

Suggested acknowledgement text follows.

> Calculations were performed using the Sulis Tier 2 HPC platform hosted by the Scientific Computing Research Technology Platform at the University of Warwick. Sulis is funded by EPSRC Grant EP/T022108/1 and the HPC Midlands+ consortium.

## Acceptable Use Policy

The Sulis acceptable use policies exist to safeguard the performance, reliability and integrity of Sulis for all users.

1. University policies and regulations
  - Sulis is subject to University of Warwick [regulations and policies](http://www2.warwick.ac.uk/services/its/policies/).
1. Support
  - All support requests should be made via the support contacts outlined on the [support page]({% link support.markdown %}). 
1. Account
  - University of Warwick regulations do not allow sharing your account. It is not possible to distinguish sharing with innocent intentions from unauthorised access by a third-party. Therefore, in order to protect your data, Sulis support staff may lock your account if they see a violation of that policy. This is for data security of your own as well as other users data.
1. Remote access
  - When using secure shell (ssh) with public/private keys, to access Sulis, the private key must be protected (encrypted) with a strong passphrase. Private keys without a passphrase are unencrypted and their use is not permitted on Sulis. Sulis support staff may: revoke/remove any unencrypted keys present on Sulis.
1. Software
  - It is not permitted for the users to install third-party, unapproved remote access tools on their accounts. Any and all remote access tools need to be installed and maintaned centrally and their installation requested via their relevant [support contacts]({% link support.markdown %}). Examples of such third party remote access software include, but aren't limited to, TeamViewer.
1. Storage
  - Users should utilise Sulis storage in accordance with the guidance provided on the [storage page](({% link gettingstarted/storage.markdown %})). Specifically Sulis storage should not be used for long term data retention since it is not backed up and to do so exposes those data to risk of being lost.
  - Users should not utilise /tmp for long term data storage; /tmp directories are subject to periodic clean-up and cannot be relied upon for anything other than temporary scratch storage.
  - All pseudo-filesystems (sshfs, encfs and other fuse-based, user-controllable filesystems) must be mounted inside /var/tmp/$USER/ where $USER is the shell variable containing the username for the current login session.
1. Login node
  - The Sulis login nodes are intended to provide remote access to Sulis. They are explicitly not intended to run compute jobs, even for relatively short periods of time or for test purposes. All compute jobs should be run through the available batch system and not on the login node directly.
  - Sulis support staff reserves the right to terminate compute processes running on login nodes without notification to the user. Please note that any late Friday or weekend revocations may not be addressed until the following Monday.
