---
layout: page
title: Path Tracer in C++
permalink: /projects/pathtracer
---

The [path tracer][pt-github] is based on the results of the university course _Global Illumination Methods_. It is written in C++ and uses OpenMP for parallelization. Some notable features are:

- Bounding-box-based hit-tests
- An Octree for the scene
- Bounding volume hierarchies (BVH) for the models in the scene
- Lambertian, metal-like and dielectric material
- Basic texturing support
- A simple obj reader

![Path tracer example output](/assets/images/path_tracer_example.jpg)

Code on [GitHub][pt-github].

<!-- URLS -->
[pt-github]:            https://github.com/JBamberger/global-illumination
