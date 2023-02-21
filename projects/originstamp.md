---
layout: page
title: Android App for Tamper-Proof Media Timestamps
permalink: /projects/originstamp
---

This application started out as a project at my university in a group of four students. Originally, the app used the free timestamping service [OriginStamp.org][os-org]. This service uses the Bitcoin blockchain to create timestamps of files. Such timestamps guarantee, that the given file existed at the point it was timestamped and was not modified since. Basically, a hash sum of the file is computed. This hash value is transformed, such that it can be used as a bitcoin address. Then a transaction is performed to the given address. Now the bitcoin blockchain contains a timestamped entry with the given address (which in turn was generated from the file hash). To prove that a file has a given timestamp, it is sufficient to compute its hash value, compute the address and then check, when the transaction was performed. For more detailed information, have a look at the [OriginStamp.org][os-org] website.

The application ended up as a commercial product of [OriginStamp.com][os-com], a startup company which provides the timestamping service at a larger scale than the academic [OriginStamp.org][os-org] variant. The application is no longer maintained by me.

- [OriginStamp Trusted Timestamping on Google Play][os-app-playstore]
- [Android client library for OriginStamp][os-client-lib]

[os-org]:               https://originstamp.org
[os-com]:               https://originstamp.com
[os-app-playstore]:     https://play.google.com/store/apps/details?id=kn.uni.isg.evidenceapp
[os-client-lib]:        https://github.com/JBamberger/originstamp-android-api
