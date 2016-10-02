---
date: 2016-10-01T21:07:13+01:00
title:  Next generation config management writen in Go.
type: index
weight: 0
---
![MGMT logo](/images/mgmt.png)

## Overview

The `mgmt` tool is a next generation config management prototype. It's not yet
ready for production, but we hope to get there soon. Get involved today!

### Why did you start this project?

I wanted a next generation config management solution that didn't have all of
the design flaws or limitations that the current generation of tools do, and no
tool existed!

## Project Description

The mgmt tool is a distributed, event driven, config management tool, that
supports parallel execution, and librarification to be used as the management
foundation in and for, new and existing software.

For more information, you may like to read some blog posts from the author:

* [Next generation config mgmt](https://ttboj.wordpress.com/2016/01/18/next-generation-configuration-mgmt/)
* [Automatic edges in mgmt](https://ttboj.wordpress.com/2016/03/14/automatic-edges-in-mgmt/)
* [Automatic grouping in mgmt](https://ttboj.wordpress.com/2016/03/30/automatic-grouping-in-mgmt/)
* [Automatic clustering in mgmt](https://ttboj.wordpress.com/2016/06/20/automatic-clustering-in-mgmt/)

There is also an [introductory video](http://meetings-archive.debian.net/pub/debian-meetings/2016/debconf16/Next_Generation_Config_Mgmt.webm) available.
Older videos and other material [is available](https://github.com/purpleidea/mgmt/#on-the-web).


## Key Features

### Parallel execution

Run all the resources concurrently (where possible)
[Read more...](/features/parallel)

### Event driven

Monitor and react dynamically only to changes (when needed)
[Read more...](/features/event_driven)

### Distributed topology

So that scale and centralization problems are replaced with a robust distributed system
[Read more...](/features/distributed)

### Puppet support

Allow you to reuse existing Puppet manifest.
[Read more...](/features/puppet)




