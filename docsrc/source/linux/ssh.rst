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

Debugging ssh
--------------

https://habr.com/en/articles/861626/

For connections over high-latency networks or when transferring large amounts of data, enabling SSH compression can yield substantial 
performance improvements. SSH compression becomes particularly effective when working with text-heavy sessions or transferring compressible data:

.. code-block:: bash

    $ cat ~/.ssh/config

    # Global SSH client configuration
    Host *
        # Enable compression for all connections
        Compression yes
        # Use compression level 6 for optimal balance
        CompressionLevel 6


Perhaps one of the most powerful optimizations involves SSH connection multiplexing. Instead of establishing new TCP connections for each SSH 
session, multiplexing reuses an existing connection, dramatically reducing connection overhead. This becomes especially valuable when working 
with remote Git repositories or running multiple SSH sessions to the same server:

.. code-block:: bash

    $ cat ~/.ssh/config
    Host *
        # Enable automatic multiplexing
        ControlMaster auto
        # Define the control socket location
        ControlPath ~/.ssh/control/%C
        # Keep the master connection alive for an hour
        ControlPersist 1h
        # Optional: Configure keepalive to prevent timeouts
        ServerAliveInterval 60
        ServerAliveCountMax 3

We can verify multiplexing is working by examining the control socket:

.. code-block:: bash

    $ ls -la ~/.ssh/control/
    total 0
    drwx------ 2 user user 100 Nov 26 10:15 .
    drwx------ 8 user user 160 Nov 26 10:15 ..
    srw------- 1 user user   0 Nov 26 10:15 example.com-22-user

For production environments where consistent performance is critical, we might also consider adjusting TCPkeepalive 
settings to prevent connection drops over problematic networks:

.. code-block:: bash

    Host production-*
        # More aggressive keepalive for production servers
        TCPKeepAlive yes
        ServerAliveInterval 30
        ServerAliveCountMax 6

