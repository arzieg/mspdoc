.. _ansible_inventory:

##########################
Ansible Inventory
##########################


Inventory File: 

.. code-block::

    susemanagerproxy:
    hosts:
        <hostname fqdn>
    vars:
        ansible_user: adminuser
        susemanager: <custom hostname>
        susemanager_fqdn: <custom fqdn>
        susemanager_ip: <custom ip>
        bootstrap: salt-bootstrap-suse-manager-proxy-43.sh

    susemanagerproxytest:
    hosts:
        <hostname fqdn>
    ...

ansible susemanagerproxy -i ../shared/<inventory> --list-hosts   -> Anzeige, welche(r) Host zu der Gruppe gehören
ansible-inventory -i ../shared/<inventory> --list                -> Anzeige der gefundenen Keys
ansible-inventory -i ../shared/<inventory> --list | jq ".[].hosts | map(.)?"   -> Anzeige der Hosts im Inventory File