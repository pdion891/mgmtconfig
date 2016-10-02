---
date: 2016-10-01T21:07:13+01:00
title:  Distributed topology
type: feature
weight: 0
description: So that scale and centralization problems are replaced with a robust distributed system.
---

All software typically runs with some sort of topology. Puppet and [Chef](https://en.wikipedia.org/wiki/Chef_%28software%29) normally run in a [client server topology](https://en.wikipedia.org/wiki/Client%E2%80%93server_model), where you typically have one server with many clients, each running an agent. They also both offer a standalone mode, but in general this is not more interesting than running a fancy bash script. In this context, I define interesting as “relating to clustered, multiple machine architectures”.

![g5](/images/graph5.png)

Here in graph g5 you can see one server which has three clients initiating requests to it.

This traditional model of computing is well-known, and fairly easy to reason about. You typically put all of your code in one place (on the server) and the clients or agents need very little personalized configuration to get working. However, it can suffer from performance and scalability issues, and it can also be a single point of failure for your infrastructure. Make no mistake: if you manage your infrastructure properly, then when your configuration management infrastructure is down, you will be unable to bring up new machines or modify existing ones! This can be a disastrous type of failure, and is one which is seldom planned for in disaster recovery scenarios!

Other systems such as Ansible are actually orchestrators, and are not technically configuration management in my opinion. That doesn’t mean they don’t share much of the same problem space, and in fact they are usually idempotent and share many of the same properties of traditional configuration management systems. They are useful and important tools!

![g6](/images/graph6.png)

The key difference about an orchestrator, is that it typically operates with a push model, where the server (or the sysadmin laptop) initiates a connection to the machines that it wants to manage. One advantage is that this is sometimes very easy to reason about for multi machine architectures, however it shares the usual downsides around having a single point of failure. Additionally there are some very real performance considerations when running large clusters of machines. In practice these clusters are typically segmented or divided in some logical manner so as to lessen the impact of this, which unfortunately detracts from the aforementioned simplicity of the solution.

Unfortunately with either of these two topologies, we can’t immediately detect when an issue has occurred and respond immediately without sufficiently advanced third party monitoring. By itself, a machine that is being managed by orchestration, cannot detect an issue and communicate back to its operator, or tell the cluster of servers it peers with to react accordingly.

The good news about current and future generation topologies is that algorithms such as the Paxos family and Raft are now gaining wider acceptance and good implementations now exist as Free Software. Mgmt depends on these algorithms to create a mesh of agents. There are no clients and servers, only peers! Each peer can choose to both export and collect data from a distributed data store which lives as part of the cluster of peers. The software that currently implements this data store is a marvellous piece of engineering called etcd.

![g7](/images/graph7.png)

In graph g7, you can see what a fully interconnected graph topology might look like. It should be clear that the numbed of connections (or edges) is quite large. Try and work out the number of edges required for a fully connected graph with 128 nodes. Hint, it’s large!

In practice the number of connections required for each peer to connect to each other peer would be too great, so instead the cluster first achieves distributed consensus, and then the elected leader picks a certain number of machines to run etcd masters. All other agents then connect through one of these masters. The distributed data store can easily handle failures, and agents can reconnect seamlessly to a different temporary master should they need to. If there is a lack or an abundance of transient masters, then the cluster promotes or demotes an agent automatically by asking it to start or stop an etcd process on its host.

![g8](/images/graph8.png)

In graph g8, you can see a tightly interconnected centre of nodes running both their configuration management tasks, but also etcd masters. Each additional peer picks any of them to connect to. As the number of nodes scale, it is far easier to scale such a cluster. Future algorithm designs and optimizations should help this system scale to unprecedented host counts. It should go without saying that it would be wise to ensure that the nodes running etcd masters are in different failure domains.

By allowing hosts to export and collect data from the distributed store, we actually end up with a mechanism that is quite similar to what Puppet calls exported resources. In my opinion, the mechanism and data interchange is actually a brilliant idea, but with some obvious shortcomings in its implementation. This is because for a cluster of N nodes, each wishing to exchange data with one another, puppet must run N times (once on each node) and then N-1 times for the entire cluster to see all of the exchanged data. Each of these runs requires an entire sequential run through every resource, and an expensive check of each resource, each time.

In contrast, with mgmt, the graph is redrawn only when an etcd event notifies us of a change in the data store, and when the new graph is applied, only members who are affected either by a change in definition or dependency need to be re-run. In practice this means that a small cluster where the resources themselves have a negligible apply time, can converge a complete connected exchange of data in less than one second.

An example demonstrates this best.

I have three nodes in the system: A, B, C.
Each creates four files, two of which it will export.
On host A, the two files are: /tmp/mgmtA/f1a and /tmp/mgmtA/f2a.
On host A, it exports: /tmp/mgmtA/f3a and /tmp/mgmtA/f4a.
On host A, it collects all available (exported) files into: /tmp/mgmtA/
It is done similarly with B and C, except with the letters B and C substituted in to the emphasized locations above.
For demonstration purposes, I startup the mgmt engine first on A, then B, and finally C, all the while running various terminal commands to keep you up-to-date.
As before, I’ve trimmed the logs and annotated the output for clarity:

```bash
james@computer:/tmp$ rm -rf /tmp/mgmt* # clean out everything
james@computer:/tmp$ mkdir /tmp/mgmt{A..C} # make the example dirs
james@computer:/tmp$ tree /tmp/mgmt* # they're indeed empty
/tmp/mgmtA
/tmp/mgmtB
/tmp/mgmtC

0 directories, 0 files
james@computer:/tmp$ # run node A, it converges almost instantly
james@computer:/tmp$ tree /tmp/mgmt*
/tmp/mgmtA
├── f1a
├── f2a
├── f3a
└── f4a
/tmp/mgmtB
/tmp/mgmtC

0 directories, 4 files
james@computer:/tmp$ # run node B, it converges almost instantly
james@computer:/tmp$ tree /tmp/mgmt*
/tmp/mgmtA
├── f1a
├── f2a
├── f3a
├── f3b
├── f4a
└── f4b
/tmp/mgmtB
├── f1b
├── f2b
├── f3a
├── f3b
├── f4a
└── f4b
/tmp/mgmtC

0 directories, 12 files
james@computer:/tmp$ # run node C, exit 5 sec after converged, output:
james@computer:/tmp$ time ./mgmt run --file examples/graph3c.yaml --hostname c --converged-timeout=5
01:52:33 main.go:65: This is: mgmt, version: 0.0.1-29-gebc1c60
01:52:33 main.go:66: Main: Start: 1452408753004161269
01:52:33 main.go:203: Main: Running...
01:52:33 main.go:103: Etcd: Starting...
01:52:33 config.go:175: Collect: file; Pattern: /tmp/mgmtC/
01:52:33 main.go:148: Graph: Vertices(8), Edges(0)
01:52:38 main.go:192: Converged for 5 seconds, exiting!
01:52:38 main.go:56: Interrupted by exit signal
01:52:38 main.go:219: Goodbye!

real    0m5.084s
user    0m0.034s
sys    0m0.031s
james@computer:/tmp$ tree /tmp/mgmt*
/tmp/mgmtA
├── f1a
├── f2a
├── f3a
├── f3b
├── f3c
├── f4a
├── f4b
└── f4c
/tmp/mgmtB
├── f1b
├── f2b
├── f3a
├── f3b
├── f3c
├── f4a
├── f4b
└── f4c
/tmp/mgmtC
├── f1c
├── f2c
├── f3a
├── f3b
├── f3c
├── f4a
├── f4b
└── f4c

0 directories, 24 files
james@computer:/tmp$
```

Amazingly, the cluster converges in less than one second. Admittedly it didn’t have large amounts of IO to do, but since those are fixed constants, it still shows how fast this approach should be. Feel free to do your own tests to verify.

