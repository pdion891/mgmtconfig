---
date: 2016-10-01T21:07:13+01:00
title:  Parallel execution
type: feature
weight: 0
description: Run all the resources concurrently (where possible).
---


Fundamentally, all configuration management systems represent the dependency relationships between their resources in a graph, typically one that is directed and acyclic.


![dependency relationships](/images/graph1.png)

Directed acyclic graph g1, showing the dependency relationships with black arrows, and the linearized dependency sort order (a topological sort) with red arrows.

Unfortunately, the execution of this graph typically has a single worker that runs through a linearized, topologically sorted version of it. There is no reason that a graph with a number of disconnected parts cannot run each separate section in parallel with each other.

![execution order of the graph](/images/graph2.png)

Graph g2 with the red arrows again showing the execution order of the graph. Please note that this graph is composed of two disconnected parts: one diamond on the left and one triplet on the right, both of which can run in parallel. Additionally, nodes 2a and 2b can run in parallel only after 1a has run, and node 3a requires the entire left diamond to have succeeded before it can execute.

Typically, some nodes will have a common dependency, which once met will allow its children to all execute simultaneously.

This is the first major design improvement that the mgmt tool implements. It has obvious improvements for performance, in that a long running process in one part of the graph (eg: a package installation) will cause no delay on a separate disconnected part of the graph which is in the process of converging unrelated code. It also has other benefits which we will discuss below.

In practice this is particularly powerful since most servers under configuration management typically combine different modules, many of which have no inter-dependencies.

An example is the best way to show off this feature. Here we have a set of four long (10 second) running processes or exec resources. Three of them form a linear dependency chain, while the fourth has no dependencies or prerequisites. I’ve asked the system to exit after it has been converged for five seconds. As you can see in the example, it is finished five seconds after the limiting resource should be completed, which is the longest running delay to complete in the whole process. That limiting chain took 30 seconds, which we can see in the log as being from three 10 second executions. The logical total of about 35 seconds as expected is shown at the end:

```bash
$ time ./mgmt run --file graph8.yaml --converged-timeout=5 --graphviz=example1.dot
22:55:04 This is: mgmt, version: 0.0.1-29-gebc1c60
22:55:04 Main: Start: 1452398104100455639
22:55:04 Main: Running...
22:55:04 Graph: Vertices(4), Edges(2)
22:55:04 Graphviz: Successfully generated graph!
22:55:04 State: graphStarting
22:55:04 State: graphStarted
22:55:04 Exec[exec4]: Apply //exec4 start
22:55:04 Exec[exec1]: Apply //exec1 start
22:55:14 Exec[exec4]: Command output is empty! //exec4 end
22:55:14 Exec[exec1]: Command output is empty! //exec1 end
22:55:14 Exec[exec2]: Apply //exec2 start
22:55:24 Exec[exec2]: Command output is empty! //exec2 end
22:55:24 Exec[exec3]: Apply //exec3 start
22:55:34 Exec[exec3]: Command output is empty! //exec3 end
22:55:39 Converged for 5 seconds, exiting! //converged for 5s
22:55:39 Interrupted by exit signal
22:55:39 Exec[exec4]: Exited
22:55:39 Exec[exec1]: Exited
22:55:39 Exec[exec2]: Exited
22:55:39 Exec[exec3]: Exited
22:55:39 Goodbye!

real    0m35.009s
user    0m0.008s
sys     0m0.008s
$
```

Note that I’ve edited the example slightly to remove some unnecessary log entries for readability sake, and I have also added some comments and emphasis, but aside from that, this is actual output! The tool also generated graphviz output which may help you better understand the problem:

 
![capability](/images/example1-dot.png)

This example is obviously contrived, but is designed to illustrate the capability of the mgmt tool.

Hopefully you’ll be able to come up with more practical examples.