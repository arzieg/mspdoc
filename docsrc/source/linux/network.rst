.. _network_allg:


#######
proxy
#######

Mal zum Testen, sshuttle: where transparent proxy meets VPN meets ssh: https://github.com/sshuttle/sshuttle/tree/master


################
IP 
################

``https://www.suse.com/de-de/support/kb/doc/?id=000019454``


Assign a Static Address

    ip a add {ip_addr/mask} dev {interface}

    Example to assign 192.168.1.200/255.255.255.0 to eth0

    ip a add 192.168.1.200/255.255.255.0 dev eth0

    OR

    ip a add 192.168.1.200/24 dev eth0
 

Bring Up The Network Interface

    ip link set dev {device}  {up|down}

    Example to bring up eth0 interface
    
    ip link set dev eth0 up


Assign a Default Gateway

    ip route add default via {gateway_addr}

    Example to set 192.168.1.254 as the default gateway

    ip route add default via 192.168.1.254
 

Assign Additional Static Routes (if needed)

    ip route add {network} via {gateway_addr}

    Example to set 192.168.1.254 as the gateway for subnet 192.168.1.0
    
    ip route add 192.168.1.0/24 via 192.168.1.254
  

At this point there should be connectivity, however if a change is needed the assigned IP address can be removed, so a new one can be used.

    ip a del {current_ip_addr} dev {device}

    Example deleting 192.168.1.200 from eth0

    ip a del 192.168.1.200 dev eth0
 

The routes can also be removed.

    ip route del {gateway_addr}
    
    OR
    
    ip route del {network} dev {device}

    Example removing default gateway 192.168.1.254
    
    ip route del 192.168.1.254
    
    Example removing 192.168.1.0 from eth0
    

#######
DNS
#######
systemctl restart nscd   - flush dns cache
