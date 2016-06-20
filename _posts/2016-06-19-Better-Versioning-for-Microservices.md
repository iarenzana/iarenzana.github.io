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

Versioning is the worst. [Semantic versioning](http://semver.org/) works well for client facing applications, but for microservices, where you might produce several production builds a week (or a day!) this method just doesn't scale. Internally, we don't necessarily care about major/minor releases and, when troubleshooting, we need some information that a version just can't get us. That's why I advocate for a dual versioning system.


Dual Versions
-

As a DevOps Engineer I need to build a bridge between development and operations. One of the bridges is needed when troubleshooting a problem with microservices. Versions matter, and git tags can do a good job; but you end up with a terrible looking git tree, full of tags all over the place and, if you have several branches, it will be a nightmare. Dual versions can be easily kept in sync; because one version is time based and the other one is Git Hash based. Both can be determined programmatically, and can be easily automated.

When running our application (imagine that we have the 'version' option), this is what we would get back:

```bash
./sqlmgr version
sqlmgr - SQL Manager
Build: 20160620.162046
Git Hash: d9beaada25cb3d505d0c1e90f222fae8a56a3753
```


Dual Versioning in Go
-
The first problem we face here, is one of the requirements that we have for versioning: automated. For some other applications, I used to do it by hand: *"When merging, just bump up the version."* I would usually say. My team and I forgot all the time and ended up with a mess that would just not scale.
Luckily, Go allows us to populate variables at link time. In our case, we have a Makefile with the build statement:

```bash
go build -ldflags "-X 'github.com/iarenzana/ittools/sqlmgr/cmd.build=`date -u +%Y%m%d.%H%M%S`' -X 'github.com/iarenzana/ittools/sqlmgr/cmd.gitHash=`git rev-parse HEAD`'"
```

Inside our `cmd` package, we can now create two empty variables (`build` and `gitHash`) and use them on the version display function. This is what the version display function in [Cobra](https://github.com/spf13/cobra) would look like.

```go

package cmd

import (
	"fmt"

	"github.com/spf13/cobra"
)

var build, gitHash string

// versionCmd represents the version command
var versionCmd = &cobra.Command{
	Use:   "version",
	Short: "Print the version number of sqlmgr",
	Long:  `All software has versions. This is sqlmgr's`,
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Printf("sqlmgr - SQL Manager\nBuild: %v\nGit Hash: %v\n", build, gitHash)
	},
}

func init() {
	RootCmd.AddCommand(versionCmd)
}
```

Very simple, right? At this point, I can create a very straight forward Makefile that will take care of all the flags for me:

``` make
build_number=`date -u +%Y%m%d.%H%M%S`
git_hash=`git rev-parse HEAD`
LDFLAGS="-X 'github.com/iarenzana/ittools/sqlmgr/cmd.build=${build_number}' -X 'github.com/iarenzana/ittools/sqlmgr/cmd.gitHash=${git_hash}'"

all:
	go build -ldflags ${LDFLAGS}
```

Advantages
-

* At a glance, a devops engineer can see when the build of the service was created and the git hash that was used. This can be compared to the git tree and easily see what's deployed.
* Fully automated. An operator doesn't need to bump up versions manually.
* The build version can be used as part of the deb/rpm name to get the packages sorted in choronological order.


Disadvantages
-

* Not user friendly. I probably wouldn't use this versioning mechanism for customer facing applications, as it doesn't give us an idea as to what to expect from the version (major vs minor, etc).


Wrap up
-

This is an idea for a new versioning system that would be very helpful for teams. Versions that can be meaningful, and useful. Versions that can speed up troubleshooting when needed.
