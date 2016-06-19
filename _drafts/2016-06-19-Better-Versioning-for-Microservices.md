---
title:  "Better Versioning for Microservices"
date:   2016-06-19 18:35:00
description: "Internal versioning for Microservices"
categoy: Tech
tags: golang development version devops
---

Better Versioning for Microservices
=

The Problem
-

Versioning is the worst. [Semantic versioning] (http://semver.org/) works well for client facing applications, but for microservices, where you might produce several production builds a week (or a day!) this method just doesn't scale. Internally, we don't necessarily care about major/minor releases and, when troubleshooting, we need some information that a version just can't get us. That's why I advocate for a dual versioning system.



Dual Versions
-

As a DevOps Engineer I need to build a bridge between development and operations. One of the bridges is needed when troubleshooting a problem with microservices. Versions matter, and git tags can do a good job; but you end up with a terrible looking git tree, full of tags all over the place and, if you have several branches, it will be a nightmare. Dual versions can be easily kept in sync; because one version is time based and the other one is Git Hash based. Both can be determined programmatically, and can be easily automated.

When running our application (imagine that we have the 'version' option), this is what we would get back:

```bash

```

Dual Versioning in Go
-
The first problem we face here, is one of the requirements that we have for versioning: automated. For some other applications, I used to do it by hand: *"When merging, just bump up the version."* I would usually say. My team and I forgot all the time and ended up with a mess that would just not scale.
Luckily, Go allows us to populate variables at link time. In our case, we have a Makefile with the build statement:

```bash
go build -o app_name
```

Inside our main package, we can now create

Advantages
-

Disadvantages
-

Wrap up
-

This is an idea for a new versioning system that would be very helpful for teams. Versions that can be meaningful, and useful. Versions that can speed up troubleshooting when needed.
