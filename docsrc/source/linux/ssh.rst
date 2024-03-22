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