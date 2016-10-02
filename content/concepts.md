---
date: 2016-10-01T21:07:13+01:00
title:  Concepts
type: page
weight: 30
---

### Autoedges

Automatic edges, or AutoEdges, is the mechanism in mgmt by which it will
automatically create dependencies for you between resources. For example,
since mgmt can discover which files are installed by a package it will
automatically ensure that any file resource you declare that matches a
file installed by your package resource will only be processed after the
package is installed.

You can read the introductory blog post about this topic here:
[https://ttboj.wordpress.com/2016/03/14/automatic-edges-in-mgmt/](https://ttboj.wordpress.com/2016/03/14/automatic-edges-in-mgmt/)


### Autogrouping

Automatic grouping or AutoGroup is the mechanism in mgmt by which it will
automatically group multiple resource vertices into a single one. This is
particularly useful for grouping multiple package resources into a single
resource, since the multiple installations can happen together in a single
transaction, which saves a lot of time because package resources typically have
a large fixed cost to running (downloading and verifying the package repo) and
if they are grouped they share this fixed cost. This grouping feature can be
used for other use cases too.

You can disable autogrouping for a resource by setting the `autogroup` key on
the meta attributes of that resource to `false`.


You can read the introductory blog post about this topic here:
[https://ttboj.wordpress.com/2016/03/30/automatic-grouping-in-mgmt/](https://ttboj.wordpress.com/2016/03/30/automatic-grouping-in-mgmt/)


### Automatic clustering

Automatic clustering is a feature by which mgmt automatically builds, scales,
and manages the embedded etcd cluster which is compiled into mgmt itself. It is
quite helpful for rapidly bootstrapping clusters and avoiding the extra work to
setup etcd.

If you prefer to avoid this feature. you can always opt to use an existing etcd
cluster that is managed separately from mgmt by pointing your mgmt agents at it
with the `--seeds` variable.

You can read the introductory blog post about this topic here:
[https://ttboj.wordpress.com/2016/06/20/automatic-clustering-in-mgmt/](https://ttboj.wordpress.com/2016/06/20/automatic-clustering-in-mgmt/)


### Remote ("agent-less") mode

Remote mode is a special mode that lets you kick off mgmt runs on one or more
remote machines which are only accessible via SSH. In this mode the initiating
host connects over SSH, copies over the `mgmt` binary, opens an SSH tunnel, and
runs the remote program while simultaneously passing the etcd traffic back
through the tunnel so that the initiators etcd cluster can be used to exchange
resource data.

The interesting benefit of this architecture is that multiple hosts which can't
connect directly use the initiator to pass the important traffic through to each
other. Once the cluster has converged all the remote programs can shutdown
leaving no residual agent.

This mode can also be useful for bootstrapping a new host where you'd like to
have the service run continuously and as part of an mgmt cluster normally.

In particular, when combined with the `--converged-timeout` parameter, the
entire set of running mgmt agents will need to all simultaneously converge for
the group to exit. This is particularly useful for bootstrapping new clusters
which need to exchange information that is only available at run time.

An introductory blog post about this topic will follow soon.
