# Build Go

## run go

go run -x main.go    // -x Anzeige von Aktivitäten, die vorher durchgeführt werden (z.B. compilieren von c libraries)


## Build pure go

* build "pure" go (with some cautions) to put it into a container:

```go
CGO_ENABLED=0 go build -a -tags netgo,osusergo -ldflags "-extldglags '-static' -s -w" -o listener .

ldd listener -> not a dynamic executable
```
netgo,osusergo = use go libaries and not shared

-static  = build static (hatte nicht funktioniert)
CGO_ENABLED = 0  -> make sure, no external C dependencies are linked
-s = strips the symbol table 
-w = remove debug information

## Cross Build

* $GOARCH defines architecture (amd64, arm64)
* $GOOS defines the os (linux, darwin)
* $GOARM for the arm chip (v7)

```go
GOOS=linux GOARCH=arm GOARM=v7 CGO_ENABLED=0 go build -a -tags netgo,osusergo -ldflags "-extldglags '-static' -w -o mainPi ./main.go

file mainPi -> ELF 32 Bit, ARM, statically linked
```

## Makefile

in main.go

```go
var version string  // do not remove or modify
...
```

Set compile-time variables for versioning

From the Makefile:

```
VERSION=$(shell git decribe --tags --long --dirty 2>/dev/null)
BRANCH=$(shell git rev-parse --abbrev-ref HEAD)

xzy: $(SOURCE)
  go build -mod=vendor -ldflags "-X main.version=$(VERSION)" -o $@ ./cmd/xyz
...
```

We can use Docker to build as well as run
* multi-stage builds
* use a golang image to build it
* copy the result to a another image

The result is a small Docker container for Linux. And you can build it without even having go installed. This is great for CI/CD environments.

Dockefile:

```
FROM golang:1.15-alpine AS builder

RUN /sbin/apk update && /sbin/apk --no-cache add ca-certificates \
git tzdata && /usr/sbin/update-ca-certificates

RUN adduser -D -g '' sort
WORKDIR /home/sort

COPY go.mod /home/sort
COPY go.sum /home/sort
COPY cmd /home/sort/cmd
COPY *.go /home/sort

ARG VERSION

RUN CGO_ENABLED=0 go build -a -tags netgo,osusergo -ldflags "-extldglags '-static' -s -w \
-ldflags "-X main.version=$VERSION" -o sort ./cmd/sort

FROM busybox:musl

COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs
COPY --from=builder /usr/share/zoneinfo /usr/share/zoneinfo
COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /home/sort/sort /home/sort

USER sort
WORKDIR /home
EXPOSE 8081

ENTRYPOINT ["/home/sort"]
```

and build it

```
docker build -t sort-anim:latest . -f build/Dockerfile --build-arg VERSION=$(VERSION)
docker tag ${NAME}:latest ${HOST}/sort-anim:${VERSION}
docker push ${HOST}/sort-anim:${VERSION}
```



## Project Setup

```
root +--- README
     +--- Makefile
     +--------------------- build/ ---- Dockerfile
     +--- cmd --- programs
     +--------------------- deploy/ --- K8s files
     +--- go.mod
     +--- go.sum
     +--- pkg --- libraries
     +--------------------- scripts/ --- miscellany
     +--- test/ --- integretation test
     +--------------------- vendor/ ---  modules
```

Ziel: möglichst schmal halten. Nicht zu tiefe Directory Strukturen

**README**
* overview - who and whar is it for
* developer setup
* project & directory structure
* dependency management
* how to build and/or install it (make targets, etc)
* how to test it (UTs, integration, end-to-end, load, etc)
* how to run it (locally, in Docker, etc)
* database & schema
* credentials & security
* debugging monitoring (metrics, logs)
* CLI tools and their usage




