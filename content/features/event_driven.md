---
date: 2016-10-01T21:07:13+01:00
title:  Event Driven
type: feature
weight: 0
description: Monitor and react dynamically only to changes (when needed).
---

All configuration management systems have some notion of idempotence. Put simply, an idempotent operation is one which can be applied multiple times without causing the result to diverge from the desired state. In practice, each individual resource will typically check the state of the element, and if different than what was requested, it will then apply the necessary transformation so that the element is brought to the desired state.

The current generation of configuration management tools, typically checks the state of each element once every 30 minutes. Some do it more or less often, and some do it only when manually requested. In all cases, this can be an expensive operation due to the size of the graph, and the cost of each check operation. This problem is compounded by the fact that the graph doesn’t run in parallel.

![sequence](/images/graph3.png)

In this time / state sequence diagram g3, time progresses from left to right. Each of the three elements (from top to bottom) want to converge on states a, b and c respectively. Initially the first two are in states x and y, where as the third is already converged. At t1 the system runs and converges the graph, which entails a state change of the first and second elements. At some time t2, the elements are changed by some external force, and the system is no longer converged. We won’t notice till later! At this time t3 when we run for the second time, we notice that the second and third elements are no longer converged and we apply the necessary operations to fix this. An unknown amount of time passed where our cluster was in a diverged or unhealthy state. Traditionally this is on the order of 30 minutes.

More importantly, if something diverges from the requested state you might wait 30 minutes before it is noticed and repaired by the system!

The mgmt system is unique, because I realized that an event based system could fulfill the same desired behaviour, and in fact it offers a more general and powerful solution. This is the second major design improvement that the mgmt tool implements.

These events that we’re talking about are inotify events for file changes, systemd events (from dbus) for service changes, packagekit events (from dbus again) for package change events, and events from exec calls, timers, network operations and more! In the inotify example, on first run of the mgmt system, an inotify watch is taken on the file we want to manage, the state is checked and it is converged if need be. We don’t ever need to check the state again unless inotify tells us that something happened!

![g4](/images/graph4.png)

In this time / state sequence diagram g4, time progresses from left to right. After the initial run, since all the elements are being continuously monitored, the instant something changes, mgmt reacts and fixes it almost instantly.

Astute config mgmt hackers might end up realizing three interesting consequences:

If we don’t want the mgmt program to continuously monitor events, it can be told to exit after the graph converges, and run again 30 minutes later. This can be done with my system by running it from cron with the --converged-timeout=1, flag. This effectively offers the same behaviour that current generation systems do for the administrators who do not want to experiment with a newer model. Thus, the current systems are a special, simplified model of mgmt!
It is possible that some resources don’t offer an event watching mechanism. In those instances, a fallback to polling is possible for that specific resource. Although there currently aren’t any missing event APIs that your narrator knows about at this time.
A monitoring system (read: nagios and friends) could probably be built with this architecture, thus demonstrating that my world view of configuration management is actually a generalized version of system monitoring! They’re the same discipline!
Here is a small real-world example to demonstrate this feature. I have started the agent, and I have told it to create three files (f1, f2, f3) with the contents seen below, and also, to ensure that file f4 is not present. As you can see the mgmt system responds quite quickly:

```bash
james@computer:/tmp/mgmt$ ls
f1  f2  f3
james@computer:/tmp/mgmt$ cat *
i am f1
i am f2
i am f3
james@computer:/tmp/mgmt$ rm -f f2 && cat f2
i am f2
james@computer:/tmp/mgmt$ echo blah blah > f2 && cat f2
i am f2
james@computer:/tmp/mgmt$ touch f4 && file f4
f4: cannot open `f4` (No such file or directory)
james@computer:/tmp/mgmt$ ls
f1  f2  f3
james@computer:/tmp/mgmt$
```

That’s fast!