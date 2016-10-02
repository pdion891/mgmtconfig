---
date: 2016-10-01T21:07:13+01:00
title:  Pkg
type: page
weight: 0
description: Manage system packages with PackageKit.
---

The pkg resource is used to manage system packages. This resource works on many different distributions because it uses the underlying packagekit facility which supports different backends for different environments. This ensures that we have great Debian (deb/dpkg) and Fedora (rpm/dnf) support simultaneously.

- deb
- rpm