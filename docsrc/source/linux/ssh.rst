.. _ssh_allg:

################
SSH
################

After sudo bash I lose my ssh keys for the ssh-agent
-------------------------------------------------------

https://serverfault.com/questions/107187/ssh-agent-forwarding-and-sudo-to-another-user

ssh-add -l ist leer, nach einem sudo bash. Um das zu vermeiden, muss das env mitgenommen werden. Dies kann in der sudoers eingestellt werden. 

.. code-block:: 
    
    visudo

    Defaults    env_keep+=SSH_AUTH_SOCK

Oder wenn man das nicht editieren möchte, kann. 

``sudo --preserve-env=SSH_AUTH_SOCK bash``


X-Forwarding
-------------

Einstellungen Client

.. code-block:: bash

    # for all hosts                                                                                                                                                                             Host *                                                                                                                                                                                        # number of seconds between null packets                                                                                                                                                    ServerAliveInterval 20
    # number of null packets to try before disconnecting
    ServerAliveCountMax 100
    ForwardAgent yes
    ForwardX11 yes
    Port 22
    StrictHostKeyChecking no
    User <username>

Einstellungen /etc/ssh/sshd_config

.. code-block:: bash

    X11Forwarding yes

xauth muss auf dem Server installiert sein. 
