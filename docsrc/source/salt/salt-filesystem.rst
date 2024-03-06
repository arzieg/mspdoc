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
    ext_pillar:
    - git:
        - __env__ file:///var/lib/salt/repos/<repo1>.git:
        - fallback: master
        - __env__ file:///var/lib/salt/repos/<repo2>.git:
        - fallback: master

    git_pillar_root: pillar

    gitfs_global_lock: False
    gitfs_remotes:
    - file:///var/lib/salt/repos/<repo1>.git
    - file:///var/lib/salt/repos/<repo2>.git

    gitfs_ref_types:
    - branch

    gitfs_saltenv:
    - LABOR:  # Configured explicitly to not have to rely on the fallback.
    - ref: master

    # Everything not listed here is not available as an environment.
    gitfs_saltenv_whitelist:
    - LABOR
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

Was erst ab Patch salt-master-3006.0-150400.8.54.1.x86_64 funktioniert hat (https://github.com/saltstack/salt/issues/65002). Hier wird das jeweilige Environment 
auch ausgecheckt, d.h. die Pillars können unterschiedlich sein in den einzelnen Branches!

.. code-block:: 

    ext_pillar:
        - git:
            - __env__ file:///var/lib/salt/repos/<repo1>.git:
            - fallback: master
            - __env__ file:///var/lib/salt/repos/<repo2>.git:
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


