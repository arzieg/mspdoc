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