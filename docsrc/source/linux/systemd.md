# systemd

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

