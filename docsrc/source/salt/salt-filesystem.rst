.. _salt_filesystem:

###############
SALT Filesystem
###############

git-integration
================

.. code-block:: bash

    fileserver_backend:
    - roots
    - git

    gitfs_provider: gitpython
    git_pillar_provider: gitpython

    ext_pillar_first: True
    # arzieg - 20240213 git_pillar_base ist der Basisbranch (steht per default auch auf master lt. doku)
    #          Zusammen mit ext_pillar kann man ein Branch auch auf git_pillar_base matchen, hier insofern als
    #          das master -> master gesetzt wird, wenn saltenv=<customEnv> gesetzt ist.
    #          Auf der Clientseite ist in der environment.conf saltenv=base gesetzt (Standard für SUSE Manager).
    #          ein salt-call pillar.items (a.k.a. saltenv=base) unterscheidet sich in der Anzahl der pillars von
    #          einem salt-call pillar.items saltenv=<customEnv>
    #          Einschränkung: ein salt-call state.highstate (saltenv=base) von der Clientseite beschwert sich über
    #          eine fehlende top.sls, es werden aber die States ausgeführt. Im SUMA funktioniert der highstate.
    #          Die Lösung funktioniert aktuell nur mit einer CustomEnv
    #          Offen: Prüfung, ob die eine allg. Lösung ist mit mehreren Environments - also G0,G1,G2...
    git_pillar_base: master
   
    ext_pillar:
        - git:
            - master file:///var/lib/salt/repos/repo1.git:
            - env: <customEnv>
            - master file:///var/lib/salt/repos/repo2.git:
            - env: <customEnv>

    git_pillar_branch: master

    git_pillar_root: pillar

    gitfs_global_lock: False
    gitfs_remotes:
    - file:///var/lib/salt/repos/repo1.git
    - file:///var/lib/salt/repos/repo2.git

    gitfs_ref_types:
    - branch

    gitfs_saltenv:
    - <customEnv>:  # Configured explicitly to not have to rely on the fallback.
    - ref: master
    # Everything not listed here is not available as an environment.
    gitfs_saltenv_whitelist:
    - <customEnv>
    - feature/.*
    - bugfix/.*



ext_pillar
-----------

.. code-block:: 

    ext_pillar:
    - git:
        - master file:///var/lib/salt/repos/<repo1>.git:
          - env: LABOR
        - master file:///var/lib/salt/repos/<repo2>.git:
          - env: LABOR


Konfiguration, welcher Branch und welches git-repo für welches (SALT)-Environment genutzt wird. In diesem Beispiel bei (pillar)env=LABOR werden 
aus repo1 und repo2 der master-Branch verwendet. Hier können nun mehrere Environments und Repos angegeben werden.

Was noch nicht funktioniert hat (https://github.com/saltstack/salt/issues/65002)

.. code-block:: 

    ext_pillar:
    - git: # global: for all accounts and envs
        - master file:///var/lib/salt/repos/<repo1>.git:
        - env: __env__
    - git: # specific: only for specific envs.
        - master file:///var/lib/salt/repos/<repo2>.git:
        - env: __env__
        - fallback: master




git_pillar_base: <branch>
--------------------------
Standard Branch der verwendet wird, wenn in ext_pillar mehrere Branches angegeben werden. 

git_pillar_branch: <branch>
-----------------------------
Wenn kein Branch angegeben wird, wird dieser hier verwendet






Helfer
--------

``salt-run fileserver.envs``    Anzeige der bekannten Environments (dies kann abweichen von den noch vorhanden Branches in git (-> siehe fileserver.update)

``salt-run fileserver.update``  Update des Fileserver Caches

``salt-run fileserver.dir_list``  Anzeige der Verzeichnisse mit Saltstates. Wenn ohne *saltenv* dann die lokalen, mit *saltenv=<env>* Anzeige des jeweiligen Environments.

``salt-run fileserver.file_list``  Anzeige der Files mit Saltstates. Wenn ohne *saltenv* dann die lokalen, mit *saltenv=<env>* Anzeige des jeweiligen Environments.

``salt-run git_pillar.update``
This will not fast-forward the git_pillar cachedir on the master. All it does is perform a git fetch. If this runner is executed with -l debug, 
you may see a log message that says that the repo is up-to-date. Keep in mind that Salt automatically fetches git_pillar repos roughly every 60 seconds 
(or whatever loop_interval is set to). 


