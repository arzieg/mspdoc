# Modules

* group packages versioned/released together
* support semantic versioning & backwards compatibility
* provide in-projet dependency management
* offer strong dependency security & availability
* continue support of vendoring  (<- wenn man code nicht in einem offenen Repo hat)
* work transparently accross the Go ecosystem

Go modules with proxying offers the value of vendoring without requring your project to vendor all the 3rd-party code in your repo

Go dependency management protects agains some risk: 
* flaky repos
* packages that disappear
* conflicting dependency versions
* surreptitious changes to public packages

"A little copying is better than a little dependency"

Literatur:
Ken Thompson, Reflections on Trusting Trust
Russ Cox, Our software dependency problem

## Compatibility rule

If an old package and a new package have the same import path, the new package must be backward compatible with the old package

An incompatible updated package should use a new URL (version). Both packages could be imported

```go
package hello

import (
    "github.com/x"
 x2 "github.com/x/v2"
)
```

## Files

go.mod   - which modules are used, has to be checked in version control 
go.sum   - checksums database, has to be checked in version control


## Environment

GOPROXY=https://proxy.golang.org,direct
GOSUMDB=sum.golang.org

Private repos
GOPRIVATE=github.com/xxx,github.com/yyy
GONOSUMDB=github.com/xxx,github.com/yyy

access to private github repos is also needed

## Maintaining dependency

Start project: `go mod init <module-name>`  // create to go.mod file

`go get -u <module>`   // get module

`go get -u ./...`      // update trasivitely

`go mod tidy`          // remove unneeded modules

`go list -m versions rsc.io/sampler`   // list availables version of a dependency


`go list -m -u all`    // List all of the modules that are dependencies of your current module, along with the latest version available for each

`go get example.com/theirmodule@v1.3.4`  // get specific version

`go get example.com/theirmodule@latest`  // get latest


## Vendoring and the local cache

Use go mod vendor to create the vendor directory; it must be in the modules root directory (along with go.mod)

Local cache: $GOPATH/pkg
* each package using a directory structure
* hash the root checksum db tree

`go clean -modcache` remove it all

## Local Packages

Macht nur bei der Entwicklung Sinn. 

```
vi go.mod
replace eval => ../eval

go get eval
```

Das Paket wird dann im anderen Package mit eval angesprochen. 

## Module release and versioning workflow
https://go.dev/doc/modules/release-workflow


