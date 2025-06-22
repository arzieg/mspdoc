# Setup GO

## install suse

zypper in go1.24
update-alternatives --config go

## install ubuntu

https://go.dev/doc/manage-install
$ go install golang.org/dl/go1.10.7@latest
$ go1.10.7 download

installiert nach $HOME/sdk/*goversion*

~/.bashrc
export GOPATH=$HOME/go
export GOBIN=$HOME/go/bin
export PATH=$PATH:$GOBIN
alias go="go1.24.4"


## Documents
https://github.com/vbd/Fieldnotes  (-> more links)
https://www.practical-go-lessons.com/
https://100go.co/   Common Go Mistakes
https://leapcell.io/blog/best-practices-design-patterns-go?ref=dailydev
https://hub.corgea.com/articles/go-lang-security-best-practices



## Projectstructure

### Beispiele
https://github.com/fortio/fortio
https://github.com/christianselig/apollo-backend
https://github.com/Tanq16/ExpenseOwl   - http/api request und server


## Projectbeispiele

https://github.com/practical-tutorials/project-based-learning

## Visual Studio Code

F1 -> Go: Install/Update Tools