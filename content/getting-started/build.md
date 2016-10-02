---
date: 2016-10-01T21:07:13+01:00
title:  Build from source
type: page
weight: 20
description: Build MGMT from github sources.
---

tl:dr

```bash
cd $GOPATH
git clone --recursive https://github.com/purpleidea/mgmt/
cd mgmt
make deps
make build
time ./mgmt run --file examples/graph0.yaml --converged-timeout=5 --tmp-prefix
```

### Build

* Make sure you have golang version 1.6 or greater installed.
* define your Golang working directory eg: ``export GOPATH=~/Documents/go``
* Clone the repository recursively, eg: ```git clone --recursive https://github.com/purpleidea/mgmt/```.
* Get the remaining golang dependencies on your own, or run ``make deps`` if you're comfortable with how we install them.
* Run ``make build`` to get a freshly built mgmt binary.
* Run ```time ./mgmt run --file examples/graph0.yaml --converged-timeout=5 --tmp-prefix``` to try out a very simple example!
* To run continuously in the default mode of operation, omit the ``--converged-timeout`` option.
* Have fun hacking on our future technology!


### Dependencies

* golang 1.6 or higher (required, available in most distros)
* golang libraries (required, available with `go get`)
   ```bash
   go get github.com/coreos/etcd/client
   go get gopkg.in/yaml.v2
   go get gopkg.in/fsnotify.v1
   go get github.com/urfave/cli
   go get github.com/coreos/go-systemd/dbus
   go get github.com/coreos/go-systemd/util
   go get github.com/coreos/pkg/capnslog
   ```

* stringer (required for building), available as a package on some platforms, otherwise via `go get`
   ```bash
   go get golang.org/x/tools/cmd/stringer
   ```

* pandoc (optional, for building a pdf of the documentation)
* graphviz (optional, for building a visual representation of the graph)

