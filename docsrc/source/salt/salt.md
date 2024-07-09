# SALT Allgemein


## SALT

```
  salt '*' test.ping
  salt '*' cmd.run 'ls -l /etc'
  salt '*' pkg.install cowsay

  salt -G 'os:Ubuntu' test.ping
  salt -E 'minion[1-9]' test.ping
  salt -L 'minion1,minion2' test.ping

  salt-key -L" will list all minions that whose public keys you've accepted on your master.
```

## Debug


```
  salt-call state.show_top                      -> welches TOP File durchläuft SALT
  salt-call state.show_states                   -> welche STATES durchläuft SALT
  salt-call state.show_sls <STATESDIR>          -> zeige Ablauf der States in <STATESDIR>
  salt-call --output=yaml state.show_sls <STATEDIR/users>  -> print yaml-files des States users.sls
  salt-call pillar.items
  salt-call pillar.items <Pillarstruktur>
  salt-call -l trace state.highstate test=True
  salt-call -l debug state.highstate test=True
```

YAML quick test: 

```
  python -c 'import yaml,sys;yaml.safe_load(sys.stdin)' < yamltest.txt
  echo $? = 0, dann ok
```

Einzelne Module aufrufen:
 `/usr/bin/salt-call --local file.dirname '/<dir>/*/user'`


## salt-state:

```
  salt '<host>' state.apply <saltstate-File>
  salt '<host>' state.apply <saltstate-File> test=True   -> only Test
  venv-salt-call state.sls_id login_timeout 01_TEMPLATE.04_XHANA2R2.CM_hardening    -> specific state id in sls file
```

vom Client: `/usr/bin/salt-call state.highstate test=True`

## pillar

Salt pillar is a system that lets you define secure data that are ‘assigned’ to one or more minions using targets. 
Salt pillar data stores values such as ports, file paths, configuration parameters, and passwords.

```
  salt '*' saltutil.refresh_pillar  -> refresh der pillar variablen
  salt '*' state.apply ftpsync pillar='{"ftpusername": "test", "ftppassword": "<passwort>"}'  -> als commandline variante
```

in salt-states wird per variablen auf pillar-werte zugegriffen:


```
    pkg.installed:
      - name: {{ pillar['editor'] }}
```

salt '<host>' pillar.items   -> get the pillar.items (server key-value)
salt '<host>  grains.items   -> get the grains.items (client key-value)


## JINJA:


Debuging JINJA:
salt 'suseminion1.pingu.box' slsutil.renderer /var/cache/salt/minion/files/base/top.sls

im SALT Code auch häufig interressant, welche Werte haben meine JINJA Variablen

```
  include:
  - .CM_ssh
  - .CM_ssh_static
  - .CM_groups
{%- set cmd = grains.get('backup_software')| upper -%}
{%- set ret = 'NETBACKUP' in cmd -%}
{%- do salt.log.error(cmd) -%}
{%- do salt.log.error(ret) -%}
{%- if 'NETBACKUP' in (grains.get('backup_software')| upper) %}
  - .CM_netbackup
{%-   endif %}
```

in der Datei /var/log/minion (wenn man auf Master auf Minion umgestellt hat), wird dann die Ausgabe geloggt, wenn man den salt-call mit -l error aufruft.
z.B. salt-call -l error state.show_states

### Ändern von Variablen in einem File 

Man kann jinja verwenden in einem config-file. Wenn man möchte, dass das auch verarbeitet wird beim salt-state, ist bei der funktion file.managed die option 
*template: jinja* mit anzugeben, ansonsten sieht man in der Datei immer nur den jinja code und nicht die Werte.




## EVENT DRIVEN INFRASTRUCTURE

`salt-run state.event pretty=True`  -> Realtime events anzeigen lassen

fire events bei einer Aktion:

```
    nano installed:
        pkg.installed:
            - name: nano
            - fire_event: True
```

`salt-call event.send /my/test/event '{"data": "my event test"}'`    -> testevent / custom event über kommandozeile

## Bacons:

Beacons let you monitor and raise events for things that are not Salt-related. The beacon system allows the minion to hook into a variety of system processes and continually monitor these processes. When monitored activity occurs in a system process, an event is sent on the Salt event bus.

Salt beacons can currently monitor and send Salt events for many system activities, including:

```
    file system changes
    system load
    service status
    shell activity, such as user login
    network and disk usage
```

Beacons are typically enabled by placing a top-level beacons section in the minion configuration file:

```
  beacons:
    inotify:
      home/user/importantfile:
        mask:
          - modify
```

ein Reactor kann auch geschrieben werden, dieser wird in salt-master unter /saltstack/etc/master dann definiert.
Format:


``` 
    <section id>:
      local.<function>:
        - tgt: <target>
        - arg:
            <arguments>

    clean_tmp:
      local.cmd.run:
        - tgt: 'os:Ubuntu'
        - expr_form: grain
        - arg:
          - rm -rf /tmp/*
```


### Praktische Probleme:

Symlink setzen: https://stackoverflow.com/questions/22673022/check-file-exists-and-create-a-symlink

Probleme mit dem Fileserver:
	`salt-run -vv fileserver.update backend=git`
	`salt-run -l debug fileserver.update backend=git`


## salt lokal


```
  zypper in git-core
  mkdir /srv/salt
  cd /srv/salt
  git config --global http.sslVerify false
  cd /srv/salt # 
    git clone <repo1>
    git clone <repo2-local>
    cp -pR <repo2-local>/* <repo1>
    cd <repo1>
    git branch -a
    git pull
	
  vi /etc/salt/minion.d/environment.conf
    saltenv: master 
    
  vi /etc/salt/minion
  file_client: local

  file_roots:
    master:
      - /srv/salt/<repo1>
  pillar_roots:
    master:
      - /srv/salt/<repo1>/pillar
```


SALT via python (https://www.tutorialspoint.com/saltstack/saltstack_python_api.htm)
http://man.hubwiz.com/docset/SaltStack.docset/Contents/Resources/Documents/docs.saltstack.com/en/latest/ref/clients/index.html

python3

.. code-block:: bash

  import salt.loader
  opts = salt.config.minion_config('/etc/salt/minion')
  grains = salt.loader.grains(opts)

salt-mine

Um mine zu disablen (wurde vom SuSE Manager verwendet: https://documentation.suse.com/external-tree/en-us/suma/4.0/suse-manager/salt/large-scale.html aber aufgrund des erzeugten Loads wieder deaktiviert)

SusE Manager hat unter /etc/salt/minion.d/_schedule folgende Datei erzeugt: 

```
  schedule:
  __mine_interval: {enabled: true, function: mine.update, jid_include: true, maxrunning: 2,
  minutes: 60, return_job: false, run_on_start: true}
```

* Wenn man das für ein System disablen möchte, dann kann man aufrufen vom salt-master:
  salt '<host>' state.sls util.mgr_mine_config_clean_up saltenv=base
  salt --batch-size 50 '*' state.sls util.mgr_mine_config_clean_up saltenv=base     (für Massenoperation in Batches)
  
Ab hier geht man davon aus, dass mine genutzt wird:

`salt-call config.get mine_functions`     - welche mine-functions sind definiert
 