# systemd

# Boot

GPT is part of the Unified Extensible Firmware Interface (UEFI) specification. Designed both for much greater disk sizes and systemic redundancy, it is larger than the legacy MBR, so it supports much larger storage devices up to 9.44 zettabytes, or 9.44 × 1021. The partition table provides pointers to up to 256 partition entries. Each entry defines a partition in the data area of the storage device.


## Target

systemctl list-dependencies multi-user.target

Default-Target ist unter /usr/lib/systemd/system/default.target ein SymLink auf ein Runlevel. Mit cat default.target kann man sich die Abhängigkeiten ansehen. Mit `systemctl get-default` kann man sich das default.target ansehen. Ändern des default.target durch Änderung des SymLink

systemctl list-units --type target   - aktuellen Zustand ansehen

## Systemd Units

|systemd Unit   |Description|
| -------- | -------- | 
|.automount     |The .automount units are used to implement on-demand (i.e., plug and play) and mounting of filesystem units in parallel during startup.|
|.device        |The .device unit files define hardware and virtual devices that are exposed to the SysAdmin in the /dev/directory. Not all devices have unit files; typically, block devices such as hard drives, network devices, and some others have unit files.|
|.mount         |The .mount unit defines a mount point on the Linux filesystem directory structure.|
|.scope         |The .scope unit defines and manages a set of system processes. This unit is not configured using unit files; rather, it is created programmatically. Per the systemd.scope man page, “The main purpose of scope units is grouping worker processes of a system service for organization and for managing resources.”|
|.service        |The .service unit files define processes that are managed by systemd. These include services such as crond cups (Common Unix Printing System), iptables, multiple logical volume management (LVM) services, NetworkManager, and more.|
|.slice          |The .slice unit defines a “slice,” which is a conceptual division of system resources that are related to a group of processes. You can think of all system resources as a pie and this subset of resources as a “slice” out of that pie.|
|.socket         |The .socket units define inter-process communication sockets, such as network sockets.|
|.swap           |The .swap units define swap devices or files.|
|.target         |The .target units define groups of unit files that define startup synchronization points, runlevels, and services. Target units define the services and other units that must be active in order to start successfully.|
|.timer          |The .timer unit defines timers that can initiate program execution at specified times.|

Anzeige über
systemctl list-unit-files -t mount
systemctl --all -t service

Manchmal gibt es aber auch bessere spezielle Befehle, z.B. systemctl list-timers (systemctl list-unit-files -t timer geht auch, ist aber nicht so schick)


## Journal
https://last9.io/blog/systemctl-logs/

| command | description | 
| -------- | -------- | 
|journalctl	                      |View all system logs in reverse chronological order.
|journalctl -u <service-name>	    |View logs for a specific service (e.g., journalctl -u nginx).
|journalctl -u nginx.service --since today | Trouble of Service nginx today
|journalctl _COMM=sshd --since="24 hours ago" | who logged in
|journalctl --since="1 hour ago" -o json-pretty | in colors
|journalctl --since="1 hour ago" --output=short-precise | exact timestamp
|journalctl --since="1 hour ago" -o verbose | magic
|journalctl --since=2024-10-29 06:30:00" --until="2024-10-29 06:33:10" | since ... until
|journalctl -f	                  |View logs in real-time (similar to tail -f, journalctl -u nginx -f).
|journalctl --since="1 hour ago"  |Last hour
|journalctl --since="10 minutes ago" | | Last 10 Minutes
|journalctl --since "YYYY-MM-DD"	|View logs from a specific date (e.g., journalctl --since "2024-12-01", journalctl --since "2024-12-01" --until "2024-12-09").
|journalctl -b	                  |View logs from the current boot session. (previous boot: journalctl -b -1)
|journalctl -b -o short-monotonic	   |View logs from the current boot session, display numbers of seconds after kernel startup
|journalctl --list-boots          |View boots
|journalctl -p <priority>    	    |Filter logs by priority (e.g., journalctl -p err for errors).
|journalctl -n <number>	          |Show the last specified number of logs (e.g., journalctl -n 100 for the last 100 logs).
|journalctl --vacuum-time=2weeks	|Clean up logs older than the specified time (e.g., two weeks).
|journalctl --vacuum-size=500M	  |Clean up logs to keep the system journal size under the specified limit.

Show Disk Usage: 
  journalctl --disk-usage
  du -sh /var/log/journal

Rotate Journal
   journalctl --rotate

Clear journal log older then x days
  journalctl --vacuum-time=2d

Restrict logs to a certain size
  journalctl --vacuum-size=100M

Restrict number of logs
  journalctl --vacuum-files=5

Config journal:
  vi /etc/systemd/journald.conf

    SystemMaxUse	Max disk space logs can take
    SystemMaxFileSize	Max size of an INDIVIDUAL log file
    SystemMaxFiles	Max number of log files

  systemctl restart systemd-journald





## systemd Timers

https://documentation.suse.com/smart/systems-management/html/systemd-working-with-timers/index.html

* systemd timer units are identified by the .timer file name extension. Each timer file requires a corresponding service file it controls
* Timers can be real-time (being triggered on calendar events) or monotonic (being triggered at a specified time elapsed from a certain starting point).
* Time units are logged to the system journal, which makes it easier to monitor and troubleshoot them.
* If the system is off during the expected execution time, the timer is executed once the system is running again.

### creating a timer

1. Create a service file (helloworld.service)
```
[Unit]
Description="Hello World script"

[Service]
ExecStart=/usr/local/bin/helloworld.sh
```

2. create a time file (helloworld.timer)
```
[Unit]
Description="Run helloworld.service 5min after boot and every 24 hours relative to activation time"

[Timer]
OnBootSec=5min
OnUnitActiveSec=24h
OnCalendar=Mon..Fri *-*-* 10:00:*
Unit=helloworld.service

[Install]
WantedBy=multi-user.target
```
3. syntax check
```
systemd-analyze verify /etc/systemd/system/helloworld.*
``` 
4. start time (systemctl start helloworld.timer)
5. enable timer (systemctl enable helloworld.timer)

### timer file

## Managing timers

**Start / stop**
```
systemctl start Timer.timer
systemctl restart Timer.timer
systemctl stop Timer.timer
```

**Enable/Disable**
```
systemctl enable Timer.timer
systemctl disable Timer.timer
```

**show content**
```
systemctl cat Timer.timer
```

**list all timers**
```
systemctl list-timers --all
systemctl list-timers PATTERN
systemctl list-timers --allPATTERN
systemctl list-timers --state=STATE (active, failed, load, sub)
```

## Timer types

### Real-time timer

Real-time timers are triggered by calendar events. They are defined using the option OnCalendar.

```
OnCalendar=Fri *-*-* 18:00:00          (on friday)
OnCalendar=Mon..Sun *-*-* 5:00:00      (every day)
OnCalendar=Tue,Sun *-*-* 01,03:00:00   (tue + sun at 01 and 03)
OnCalendar=Mo..Sun 2023-09-23 00:00:01 (single date)

Multiple times ( you can create more than one OnCalendar entry in a single timer file):
OnCalendar=Mon..Fri *-*-* 10:00  
OnCalendar=Sat,Sun *-*-* 22:00

see man 7 systemd.time
```

### Monotonic timers

Monotonic timers are triggered at a specified time elapsed from a certain event, such as a system boot or system unit activation event.

```
OnActiveSec=50minutes
OnBootSec=10hours
OnStartupSec=5minutes 20seconds
OnUnitInactiveSec=2hours 15minutes 18 seconds
```

| command | description | 
| -------- | -------- | 

| Timer    | Monotonic  | Definition |
| -------- | ---------- | ---------- |
| OnActiveSec= | X      | This defines a timer relative to the moment the timer is activated. |
| OnBootSec=   | X      | This defines a timer relative to when the machine boots up. |
| OnStartupSec= | X     | This defines a timer relative to when the service manager first starts. For system timer units, this is very similar to OnBootSec=, as the system service manager generally starts very early at boot. It’s primarily useful when configured in units running in the per-user service manager, as the user service manager generally starts on first login only, not during boot. |
| OnUnitActiveSec=| X   | This defines a timer relative to when the timer that is to be activated was last activated. |
| OnUnitInactiveSec=| X | This defines a timer relative to when the timer that is to be activated was last deactivated. |
| OnCalendar=  |   | This defines real-time (i.e., wall clock) timers with calendar event expressions. See systemd.time(7) for more information on the syntax of calendar event expressions. Otherwise, the semantics are similar to OnActiveSec= and related settings. This timer is the one most like those used with the cron service. |

### Transient timers

Transient timers are temporary timers that are only valid for the current session. Using these timers, you can either use an existing service file or start a program directly. Transient timers are invoked by running systemd-run

```
sudo systemd-run --on-active="2hours" --unit="helloworld.service"  (run every 2 hours)
sudo systemd-run --on-active="2hours" /usr/local/bin/helloworld.sh (run directly)
sudo systemd-run --on-active="2hours" /usr/local/bin/helloworld.sh --language=pt_BR
```

Transient timers can be monotonic or real-time. The following switches are supported and work as described in Monotonic timers:
* --on-active
* --on-startup
* --on-unit-active
* --on-unit-inactive
* --on-calendar

## Testing calendar

systemd provides a tool for testing and creating calendar timer entries for real-time timers: **systemd-analyze calendar**

```
systemd-analyze calendar "Tue,Sun *-*-* 01,03:00:00"
Normalized form: Tue,Sun *-*-* 01,03:00:00
Next elapse: Sun 2021-10-31 01:00:00 CEST
(in UTC): Sat 2021-10-30 23:00:00 UTC
From now: 3 days left

systemd-analyze calendar "Mon..Fri *-*-* 10:00" "Sat,Sun *-*-* 22:00"
Original form: Mon..Fri *-*-* 10:00
Normalized form: Mon..Fri *-*-* 10:00:00
Next elapse: Thu 2021-10-28 10:00:00 CEST
(in UTC): Thu 2021-10-28 08:00:00 UTC
From now: 19h left

Original form: Sat,Sun *-*-* 22:00
Normalized form: Sat,Sun *-*-* 22:00:00
Next elapse: Sat 2021-10-30 22:00:00 CEST
(in UTC): Sat 2021-10-30 20:00:00 UTC
From now: 3 days left
```

```
systemd-analyze calendar --iterations 5 "Sun *-*-* 0/08:00:00"
Original form: Sun *-*-* 0/08:00:00
Normalized form: Sun *-*-* 00/8:00:00
Next elapse: Sun 2021-10-31 00:00:00 CEST
(in UTC): Sat 2021-10-30 22:00:00 UTC
From now: 3 days left
Iter. #2: Sun 2021-10-31 08:00:00 CET
(in UTC): Sun 2021-10-31 07:00:00 UTC
From now: 3 days left
Iter. #3: Sun 2021-10-31 16:00:00 CET
(in UTC): Sun 2021-10-31 15:00:00 UTC
From now: 4 days left
Iter. #4: Sun 2021-11-07 00:00:00 CET
(in UTC): Sat 2021-11-06 23:00:00 UTC
From now: 1 week 3 days left
Iter. #5: Sun 2021-11-07 08:00:00 CET
(in UTC): Sun 2021-11-07 07:00:00 UTC
From now: 1 week 3 days left
```

## EMail notification

-> siehe Webseite

## using timers as a regular user

systemd timers can also be used by regular users. It helps you to automate recurring tasks like backups, processing images, or moving data to the cloud.

The same procedures and tasks as for system-wide timers are valid. However, the following differences apply:

* Timer and service files must be placed in ~/.config/systemd/user/.
* All systemctl and journalctl commands must be run with the --user switch. 
```
systemctl --user start ~/.config/systemd/user/helloworld.timer
systemctl --user enable ~/.config/systemd/user/helloworld.timer
systemctl --user list-timers
journalctl --user -u helloworld.*
systemd-analyze verify ~/.config/systemd/user/helloworld.timer
```

**Important:**

User timers only run during an active session
Instead, to start user timers at boot time and keep them running after logout, enable lingering for each affected user:
```
sudo loginctl enable-linger USER
```

Important: Environment variables are not inherited
To check the systemd environment, run systemctl --user show-environment
To import any variables missing in the systemd environment, specify the following command at the end of your ~/.bashrc:
```
systemctl --user import-environment VARIABLE1 VARIABLE2
```

## Migrating from cron to systemd timers

-> siehe Weblink

## Troubleshoot

### Check system journal for errors

``` 
sudo journalctl -u  helloworld.* 
```
* -b: Only show entries for the current boot.
* -S today: Only show entries from today.
* -x: Show help texts alongside the log entry.
* -f: Start with the most recent entries and continuously print the log as new entries get added. Useful to check triggers that occur in short intervals. Exit with Ctrl–C.

### catching up on missed runs

If a systemd timer was inactive or the system was off during the expected execution time, missed events can optionally be triggered immediately when the timer is activated again. To enable this, add the configuration option Persistent=true to the [Timer] section:

``` 
[Timer]
OnCalendar=Mon..Fri 10:00
Persistent=true
Unit=helloworld.service
```


## Mount (via systemd)

Mount units may be configured either with the traditional /etc/fstab file or with systemd units. Fedora uses the fstab file as it is created during the installation. However, systemd uses the systemd-fstab-generator program to translate the fstab file into systemd units for each entry in the fstab file. Now that you know you can use systemd .mount unit files for filesystem mounting, let’s create a mount unit for the new filesystem.

Create /etc/systemd/system/TestFS.mount

```
# This mount unit is for the TestFS filesystem
[Unit]
Description=TestFS Mount
[Mount]
What=/dev/vdb
Where=/TestFS
Type=ext4
Options=defaults
[Install]
WantedBy=multi-user.target
```

```
systemd-analyze verify /etc/systemd/system/TestFS.mount 
systemctl enable TestFS.mount 
systemctl start TestFS.mount
systemctl status TestFS.mount
```


