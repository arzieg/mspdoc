# Delve

## Installation

`go get github.com/go-delve/delve/cmd/dlv`

## General

https://github.com/go-delve/delve/blob/master/Documentation/README.md

Specify target and start debugging with the default terminal interface:
dlv debug \[package\]
dlv test \[package\]
dlv exec \<exe\>
dlv attach \<pid\>
dlv core \<exe\> \<core\>
dlv replay \<rr trace\>

## Environmentvariables

$DELVE_EDITOR is used by the edit command (if it isn't set the $EDITOR variable is used instead)
$DELVE_PAGER is used by commands that emit large output (if it isn't set the $PAGER variable is used instead, if neither is set more is used)
$TERM is used to decide whether or not ANSI escape codes should be used for colorized output

## Basic commands

help
list main.main        (show source code by package und function name)
list ./main.go:14     (show source code by line)
funcs fib             (find function fib, list with list main.fib)

## Breakpoints




