---
layout: page
title: Projects
permalink: /projects/
---
Here is a short list of things I worked on which are available for usage:

## Andorid app to display class substitutions

I developed this app in my time at the highschool. The school had a website which lists current substitutions and changes to the course schedule. This website was very inconvenient for mobile devices, therefore I decided to develop an application which can display this plan more readable.
In addition, the application allows the user to set its own courses, such that all irrelevant changes are filtered out.
Later on I added some functions, for example a screen to view the schools blog directly inside of the app.

- [FHG Application on Github](https://github.com/JBamberger/fhg-android-app)
- [FHG Application on Google Play](https://play.google.com/store/apps/details?id=xyz.jbapps.vplan)

## Andorid app for tamperproof media timestamps

This application started out as a project at my university in a group of four students. Originally the app used the free timestamping service [OriginStamp.org](https://originstamp.org/). This service uses the bitcoin blockchain to create timestamps of files. Such timestamps guarantee, that the given file existed at the point it was timestamped and was not modified since. Basically, a hash sum of the file is computed. This hash value is transformed, such that it can be used as a bitcoin address. Then a transaction is performed to the given address. Now the bitcoin blockchain contains a timestamped entry with the given address (which in turn was generated from the file hash). To prove, that a file has a given timestamp it is sufficient to compute its hash value, compute the address and then check, when the transaction was performed. For more detailed information, have a look at the [originstamp.org](https://originstamp.org/) website.

The application ended up as a commercial product of [OriginStamp.com](https://orginstamp.com/), a startup company which provides the timestamping service at a larger scale than the _org_ variant and is no longer maintained by me.

- [OriginStamp Trusted Timestamping on Google Play](https://play.google.com/store/apps/details?id=kn.uni.isg.evidenceapp)
