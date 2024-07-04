# SBD

## Inquisitor

Kill des Knoten, wenn watchdog-timer nicht zurückgesetzt wurde.

```ps aux |grep -e COMMAND -e „sbd: inquisitor“ | grep -v grep```

## sbd disk watchers

```ps aux | grep -e COMMAND -e "sbd: watcher: /dev" |grep -v grep```

## sbd health and quorum watchers

```ps aux | grep -e COMMAND -e "sbd: watcher: Pacemaker" -e "sbd: watcher: Cluster" |grep -v grep```

