---
layout: page
title: Projects
permalink: /projects/
---

## Path tracer in C++

The [application][pt-github] is based on the results of the university course _Global Illumination Methods_. The
path tracer is written in C++ and uses OpenMP for parallelization. Some notable features are:

- Bounding-box-based hit-tests
- An Octree for the scene
- Bounding volume hierarchies (BVH) for the models in the scene
- Lambertian, metal-like and dielectric material
- Basic texturing support
- A simple obj reader

![Path tracer example output](/assets/images/path_tracer_example.jpg)

## Android app to display class substitutions

I developed this app during the time at [highschool][fhg-website]. The school had a website which lists current substitutions and changes to the course schedule. At the time, this website was very inconvenient for mobile devices, therefore I decided to develop an application which can display this plan more readable.
In addition, the application allows the user to set its own courses, such that all irrelevant changes are filtered out.
Later on I added some functions to show other content from the schools web site, for example the schools blog posts, directly inside of the app.

- [FHG Application on Github][fhg-app-github]
- [FHG Application on Google Play][fhg-app-playstore]
- [FHG Application Website][fhg-app-website]

## Android app for tamper-proof media timestamps

This application started out as a project at my university in a group of four students. Originally, the app used the free timestamping service [OriginStamp.org][os-org]. This service uses the Bitcoin blockchain to create timestamps of files. Such timestamps guarantee, that the given file existed at the point it was timestamped and was not modified since. Basically, a hash sum of the file is computed. This hash value is transformed, such that it can be used as a bitcoin address. Then a transaction is performed to the given address. Now the bitcoin blockchain contains a timestamped entry with the given address (which in turn was generated from the file hash). To prove that a file has a given timestamp, it is sufficient to compute its hash value, compute the address and then check, when the transaction was performed. For more detailed information, have a look at the [OriginStamp.org][os-org] website.

The application ended up as a commercial product of [OriginStamp.com][os-com], a startup company which provides the timestamping service at a larger scale than the academic [OriginStamp.org][os-org] variant. The application is no longer maintained by me.

- [OriginStamp Trusted Timestamping on Google Play][os-app-playstore]
- [Android client library for OriginStamp][os-client-lib]

[fhg-website]:          https://fhg-radolfzell.de
[fhg-app-playstore]:    https://play.google.com/store/apps/details?id=xyz.jbapps.vplan
[fhg-app-github]:       https://github.com/JBamberger/fhg-android-app
[fhg-app-website]:       https://www.jbamberger.de/fhg-android-app/

[os-org]:               https://originstamp.org
[os-com]:               https://originstamp.com
[os-app-playstore]:     https://play.google.com/store/apps/details?id=kn.uni.isg.evidenceapp
[os-client-lib]:        https://github.com/JBamberger/originstamp-android-api

[pt-github]:            https://github.com/JBamberger/global-illumination
