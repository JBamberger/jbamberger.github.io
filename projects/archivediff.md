---
layout: page
title: Archive Diff Tool
permalink: /projects/archivediff
---

A tool to compare archives of various file types, e.g. tar, zip or directories. The main use-case
for this tool was to clean up various compressed archives used as backup. Since the file types
differed between the archives it was not feasible to simply use the diffing capabilities of tools
like `tar`.

The tool is written in Python and computes the diff by comparing file hashes. It also includes a
function to strip prefix paths from the output, i.e. the leading directory name present for all
files. For most archive types no explicit extraction of the entire archive is necessary, as the
files can be read one-by-one. This lends itself to efficient in-memory processing.

Code on [GitHub](https://github.com/JBamberger/archive-diff).
