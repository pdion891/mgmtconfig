---
date: 2016-10-01T21:07:13+01:00
title:  File
type: page
weight: 0
description: Manage files and directories.
---


The file resource manages files and directories. In `mgmt`, directories are
identified by a trailing slash in their path name. File have no such slash.

| Properties | Description  |
| -----------|-------------|
| `Path` | The path property specifies the file or directory that we are managing. |
| `Content` | The content property is a string that specifies the desired file contents. |
| `Source` | The source property points to a source file or directory path that we wish to copy over and use as the desired contents for our resource. |
| `State` | The state property describes the action we'd like to apply for the resource. The possible values are: `exists` and `absent`. |
| `Recurse` | The recurse property limits whether file resource operations should recurse into and monitor directory contents with a depth greater than one. |
| `Force` | The force property is required if we want the file resource to be able to change a file into a directory or vice-versa. If such a change is needed, but the force property is not set to `true`, then this file resource will error. |
