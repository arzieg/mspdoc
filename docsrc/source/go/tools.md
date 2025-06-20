# Tools



## Links
https://medium.com/@stallone119/boost-your-golang-development-efficiency-with-these-tools-eb8ccc722c7d

## Local Development Environment: ServBay

When to Use ServBay?

* Need to quickly set up a Golang + MySQL/PostgreSQL/Redis environment on your local machine
* Work on multiple projects spanning different stacks (Golang + PHP + Node.js) and require easy version switching
* Need HTTPS/SSL certificates but don’t want to manually configure Nginx

ServBay eliminates tedious manual setup, letting you focus on coding rather than environment issues.
https://www.servbay.com/

## gofmt
will put your code in standard form

## goimports
like gofmt and also update import lists

## Code Quality Checking: golangci-lint

https://golangci-lint.run/

* exported names should have comments for godoc
* names shouldn't have under_scores or be in ALLCAPS
* names shouldn't be used for normal error handling
* the error flow should be indented, the happy path not
* variable declarations shouldn't have redundant type info

(all bases on Effective Go and Google's Go Code Review Comments)

can be configured with 
.golangci.yml

False positivescan be marked with **//nolint**


## Go vet

will find some issues the compiler won't
* suspicious printf format strings
* accidentally copying a mutex type
* possibly invalid integer shifts
* possibly invalid atomic assignments
* possibly invalid struct tags
* unreachable code

## Other tools

goconst - finds literals that should be declared with const
gosec   - looks for possible security issues
ineffasign - finds assignments thar are ineffective (shadowed?), f.i. err assign and do not check and in the next row err is assign again
gocyclo - reports high cyclomatic complexity in functions - complexity of a function, if to high should be break it down
deadcode, unused and varcheck - find unused/dead code
unconvert - finds redundant type conversions


## Dependency Management: Go Modules

https://go.dev/ref/mod


# Development

https://github.com/air-verse/air  - Live reload for go apps (immer wenn bei der Entw. etwas geändert wird, wird neues binary erzeugt und gestartet)
https://github.com/go-chi/chi     - chi is a lightweight, idiomatic and composable router for building Go HTTP services.
https://github.com/go-gorm/gorm   - The fantastic ORM library for Golang, aims to be developer friendly.
https://github.com/stephenafamo/bob - Bob is a set of Go packages and tools to work with SQL databases.
https://github.com/jmoiron/sqlx   - sqlx is a library which provides a set of extensions on go's standard database/sql library



# Lebensweisheiten

* clear is better than clever
* a little copying is better than a little dependency
* concurrency is not parallism
* channels orchestrate; mutex serialize
* don't communicate by sharing memory, share memory by communicating
* make the zero value useful
* the bigger the interface, the weaker the abstraction
* errors are values
* don't just check errors, handle them gracefully

## video recommedations

* simple made easy (Rick Hickey, QCon 2012)
* small is beautiful (Kevlin Henney, goto 2016)
* software that fits in your head (Dan North, goto 2016)
* worse is better, for better or for worse (Kevlin Henney, goto 2013)
* solid snakes ot how to take 5 weeks of vacation (Hynek Schlawack, pycon 2017)

(https://github.com/matt4biz/go-resources)


## Debug

https://github.com/goforj/godump
